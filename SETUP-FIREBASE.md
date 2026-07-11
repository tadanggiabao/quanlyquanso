# Thiết lập Firebase cho Sổ Quân Số (bản có đăng nhập, đồng bộ thời gian thực)

Làm theo đúng thứ tự các bước. Toàn bộ đều miễn phí ở quy mô một đại đội (gói Firebase
"Spark" miễn phí đủ dùng thoải mái).

## Bước 1 — Tạo project Firebase
1. Vào **console.firebase.google.com**, đăng nhập bằng tài khoản Google.
2. Chọn **Add project** (Thêm dự án) → đặt tên, ví dụ `so-quan-so` → bỏ qua Google
   Analytics (không cần) → **Create project**.

## Bước 2 — Bật đăng nhập Email/Mật khẩu
1. Trong menu bên trái: **Build → Authentication** → **Get started**.
2. Tab **Sign-in method** → chọn **Email/Password** → bật **Enable** → **Save**.

## Bước 3 — Tạo cơ sở dữ liệu Firestore
1. Menu bên trái: **Build → Firestore Database** → **Create database**.
2. Chọn vị trí server gần bạn nhất (ví dụ `asia-southeast1`) → chọn chế độ
   **Start in production mode** → **Enable**.
3. Vào tab **Rules** ngay trong Firestore → xoá hết nội dung mặc định → dán toàn bộ
   nội dung file **`firestore.rules`** đi kèm gói này vào → **Publish**.
   (File này quy định: đại đội trưởng thấy/sửa tất cả, trung đội trưởng chỉ thấy đúng
   trung đội mình, và chặn hoàn toàn người lạ.)

## Bước 4 — Lấy thông tin cấu hình (config) để dán vào app
1. Menu bên trái, chọn icon **⚙️ Project settings**.
2. Kéo xuống mục **Your apps** → chọn biểu tượng **</>** (Web) → đặt tên bất kỳ, ví dụ
   `so-quan-so-web` → **Register app** (không cần Firebase Hosting).
3. Firebase sẽ hiện ra một đoạn code chứa `firebaseConfig = {...}` — copy toàn bộ
   6 dòng bên trong (`apiKey`, `authDomain`, `projectId`, `storageBucket`,
   `messagingSenderId`, `appId`).
4. Mở file **`index.html`** bằng trình soạn thảo bất kỳ, tìm đến đoạn:
   ```js
   const firebaseConfig = {
     apiKey: "DÁN_API_KEY_CỦA_BẠN",
     ...
   };
   ```
   (nằm gần đầu thẻ `<script type="module">`) → thay bằng đoạn bạn vừa copy ở bước 3.

## Bước 5 — Tạo tài khoản Đại đội trưởng đầu tiên (thủ công, chỉ làm 1 lần)
App chỉ cho phép **đại đội trưởng** tạo thêm tài khoản khác — nên tài khoản đại đội
trưởng *đầu tiên* phải tạo tay trong Firebase Console:

1. **Authentication → Users → Add user** → nhập email + mật khẩu cho đại đội trưởng
   → **Add user**. Sau khi tạo, bấm vào dòng user đó và **copy đoạn User UID**
   (chuỗi ký tự dài).
2. Sang **Firestore Database → Data → Start collection**:
   - Collection ID: `users`
   - Document ID: **dán đúng User UID** vừa copy ở trên
   - Thêm các trường (field):
     | Trường | Kiểu | Giá trị |
     |---|---|---|
     | `role` | string | `commander` |
     | `name` | string | Tên đại đội trưởng, ví dụ `Nguyễn Văn A` |
     | `email` | string | email vừa tạo |
   - Bấm **Save**.

Vậy là xong tài khoản gốc. Từ giờ, đăng nhập bằng email/mật khẩu này sẽ vào được
toàn bộ ứng dụng với quyền đại đội trưởng, và có thể tự tạo thêm tài khoản trung đội
trưởng ngay trong app (menu ⋮ → "Cấp tài khoản mới") — không cần đụng vào Firebase
Console nữa.

## Bước 6 — Đưa app lên mạng và cài lên điện thoại
Giống hướng dẫn cũ: đưa 5 file (`index.html`, `manifest.json`, `service-worker.js`,
`icons/`) lên GitHub Pages hoặc Netlify Drop, rồi mở link đó trên điện thoại →
"Thêm vào Màn hình chính". Xem chi tiết trong `HUONG-DAN.md`.

## Cách hoạt động phân quyền (tóm tắt)
- **Đại đội trưởng**: đăng nhập → thấy toàn bộ quân số mọi trung đội, thêm/sửa/xoá
  được, đổi tên đơn vị, và cấp/thu hồi tài khoản trung đội trưởng.
- **Trung đội trưởng**: đăng nhập → chỉ thấy quân nhân có trường "Trung đội" trùng
  khớp với trung đội được gán cho mình — không có nút thêm/sửa/xoá. Việc này được
  chặn ở **cả 2 lớp**: giao diện ẩn nút, và Firestore Rules từ chối mọi request đọc
  dữ liệu ngoài phạm vi trung đội — nên dù ai đó cố tình sửa code trình duyệt cũng
  không lấy được dữ liệu trung đội khác.
- Khi đại đội trưởng cấp tài khoản mới cho trung đội trưởng, phải nhập đúng **tên
  Trung đội** khớp chính xác (phân biệt hoa/thường, khoảng trắng) với tên đã dùng khi
  nhập hồ sơ quân nhân — nên cứ gõ thống nhất một cách, ví dụ luôn là `Trung đội 1`,
  `Trung đội 2`...

## Xử lý sự cố thường gặp
- **"Tài khoản chưa được cấp quyền truy cập"** khi đăng nhập → tài khoản đó chưa có
  document tương ứng trong collection `users`, hoặc đại đội trưởng chưa cấp/đã thu hồi.
- **Trung đội trưởng đăng nhập nhưng không thấy ai** → tên Trung đội trong tài khoản
  không khớp tuyệt đối với trường "Trung đội" trong hồ sơ quân nhân.
- **Lỗi "Missing or insufficient permissions"** → thường do chưa dán đúng nội dung
  `firestore.rules` ở Bước 3, hoặc quên bấm **Publish**.
