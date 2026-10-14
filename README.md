# MedMinder — Ứng dụng nhắc uống thuốc & theo dõi sức khỏe

Dự án Android Studio (Kotlin + Jetpack Compose + Room) cho đồ án môn Lập trình điện thoại di động — Nhóm 9.

## Chức năng AI (Gemini)

Dự án có 3 chức năng AI: đọc đơn thuốc, phân tích sức khỏe, trợ lý hỏi đáp. Cần thêm `GEMINI_API_KEY` vào `local.properties`.
Xem hướng dẫn đầy đủ trong **HUONG_DAN_AI.md**.

## Cách mở dự án

1. Giải nén file zip.
2. Mở **Android Studio** (bản Koala trở lên) → **Open** → chọn thư mục `MedMinder` vừa giải nén.
3. Android Studio sẽ tự tạo Gradle Wrapper khi build lần đầu (cần kết nối mạng để tải dependency từ Google/Maven).
4. Chọn thiết bị ảo (API 26 trở lên) → bấm **Run**.

## Cấu trúc thư mục

```
app/src/main/java/com/nhom9/medminder/
├── MedMinderApp.kt          # Application: khởi tạo Room DB, repository, notification channel
├── MainActivity.kt          # Điểm vào app, xin quyền thông báo, nạp NavGraph
├── data/
│   ├── entity/               # 11 Entity Room — dịch trực tiếp từ QL_UongThuocDungGio.sql
│   ├── dao/                  # DAO tương ứng từng bảng + các câu JOIN cho Trang chủ/Lịch sử
│   ├── repository/           # ThuocRepository, ChiSoSucKhoeRepository
│   └── AppDatabase.kt
├── reminder/
│   ├── AlarmScheduler.kt          # Đặt/hủy báo thức bằng AlarmManager
│   ├── ReminderBroadcastReceiver.kt  # Hiện thông báo đúng giờ
│   ├── ReminderActionReceiver.kt     # Xử lý nút "Đã uống" / "Bỏ qua"
│   └── BootReceiver.kt               # Đặt lại lịch sau khi khởi động máy
└── ui/
    ├── theme/                 # Màu sắc, Typography, Theme.kt
    ├── navigation/NavGraph.kt # Điều hướng toàn bộ 8 màn hình
    └── screens/               # 1 package con cho mỗi màn hình (screen + viewmodel)
```

## Bảng dữ liệu (Room) khớp với file SQL của nhóm

| Bảng trong SQL | Entity Kotlin | Trạng thái |
|---|---|---|
| VaiTro | `VaiTro.kt` | Đầy đủ |
| NguoiDung | `NguoiDung.kt` | Đầy đủ |
| LienKetNguoiThan | `LienKetNguoiThan.kt` | Đầy đủ |
| Thuoc | `Thuoc.kt` | Đầy đủ, có UI CRUD |
| LichUongThuoc | `LichUongThuoc.kt` | Đầy đủ, có UI |
| LichSuUong | `LichSuUong.kt` | Đầy đủ, có UI |
| ChiSoSucKhoe | `ChiSoSucKhoe.kt` | Đầy đủ, có UI |
| NguongCanhBao | `NguongCanhBao.kt` | Đầy đủ, có UI đặt ngưỡng + cảnh báo |
| ThongBao | `ThongBao.kt` | Đầy đủ, có màn Thông báo + chấm đỏ |
| CauHinhNhac | `CauHinhNhac.kt` | Đầy đủ, có màn Cấu hình nhắc |
| CauHinhHeThong | `CauHinhHeThong.kt` | Có DAO, chưa dùng trên UI |

## Tài khoản dùng để thử app

| Vai trò | Email | Mật khẩu | Ghi chú |
|---|---|---|---|
| Admin (có sẵn) | `admin@medminder.vn` | `admin123` | Tự tạo khi mở app lần đầu. Vào màn **Quản trị hệ thống**. |
| Người dùng demo | `demo@medminder.vn` | (không có) | Chỉ vào được bằng nút **"Dùng thử không cần tài khoản"**. |
| Người dùng thường | tự đăng ký | tự đặt (≥ 6 ký tự) | Màn **Đăng ký** chỉ tạo tài khoản người dùng; **không thể tự đăng ký Admin**. |

Mã nguồn khu vực Admin nằm ở `ui/screens/admin/`. Quyền của Admin:

- **Trang chủ Admin**: thống kê toàn hệ thống (số người dùng, tài khoản bị khóa, số loại thuốc, tỉ lệ tuân thủ chung, tổng lượt
  đã uống / bỏ qua / bỏ lỡ). Bấm vào thẻ **Đã uống / Bỏ qua / Bỏ lỡ** để xem từng lượt của mọi người dùng.
  Có danh sách **người dùng cần chú ý** (bỏ lỡ nhiều liều nhất) và đổi mật khẩu của chính Admin.
- **Nhật ký uống thuốc toàn hệ thống** (`AdminNhatKyScreen`): xem ai uống / bỏ qua / bỏ lỡ thuốc gì, lúc nào; lọc theo
  trạng thái; tìm theo tên, email hoặc tên thuốc (500 lượt gần nhất).
- **Quản lý người dùng**: mỗi người hiển thị số lượt đã uống / bỏ qua / bỏ lỡ và % tuân thủ; tìm kiếm; lọc theo vai trò
  hoặc "Bị khóa"; sắp xếp "Bỏ lỡ nhiều nhất"; **tạo tài khoản** (bệnh nhân / người thân / admin); khóa / mở khóa.
- **Chi tiết người dùng**: thông tin, thống kê tuân thủ, lịch sử uống gần đây, thuốc, chỉ số sức khỏe, liên kết người thân;
  **đổi vai trò** (bệnh nhân ↔ người thân), **đặt lại mật khẩu** (về `123456`), **xóa tài khoản**
  (không xóa được chính mình, Admin khác và tài khoản demo).
- Tài khoản đang đăng nhập mà bị khóa / xóa sẽ tự bị đăng xuất.

## Chức năng người dùng

- Đăng ký / đăng nhập / đăng xuất / giữ phiên / dùng thử; **chỉnh sửa hồ sơ** và **đổi mật khẩu**.
- Thêm / sửa / xóa thuốc, nhiều giờ nhắc mỗi thuốc, **chọn các ngày trong tuần** và **ngày kết thúc** của đợt dùng thuốc.
- Chụp ảnh đơn thuốc + OCR (ML Kit) để gợi ý tên thuốc.
- Báo thức đúng giờ (AlarmManager), nút "Đã uống / Bỏ qua" trên thông báo, đặt lại cho ngày hợp lệ kế tiếp, đặt lại sau khi khởi động máy.
- Tự đánh dấu "bỏ lỡ" sau X phút (cấu hình được), âm thanh + rung; cảnh báo người thân.
- Lịch sử uống thuốc, xuất báo cáo PDF.
- Chỉ số sức khỏe, biểu đồ, ngưỡng cảnh báo.
- Mời người thân, chấp nhận / từ chối, xem lịch sử bệnh nhân.
- **Thêm thuốc bằng giọng nói** (tiếng Việt): nút "Nói để thêm thuốc" ở màn Thêm thuốc.
- **Đăng nhập bằng khuôn mặt / vân tay**: bật trong Cá nhân & Cài đặt, dùng sinh trắc học đã đăng ký trên điện thoại.
- Giao diện sáng / tối, chế độ chữ lớn.

## Hạn chế đã biết (hướng phát triển)

- Dữ liệu chỉ lưu cục bộ (Room/SQLite): chưa có Firebase Authentication / Firestore, chưa đồng bộ nhiều máy.
- "Cảnh báo người thân" được ghi thành `ThongBao` trong CSDL của chính máy này (người thân thấy khi đăng nhập trên cùng máy),
  chưa đẩy thông báo sang máy khác (cần FCM).
- Mật khẩu băm SHA-256 (chưa salt) — đủ cho bản demo học tập.
- Chưa có: theo dõi số lượng thuốc còn lại, nhắc tái khám, tìm kiếm/lọc lịch sử, "quên mật khẩu" tự phục vụ (hiện do Admin đặt lại).

## Ghi chú về AlarmManager

`SCHEDULE_EXACT_ALARM` là quyền nhạy cảm trên Android 12+. Ứng dụng đã khai báo trong Manifest; khi chạy trên máy thật/máy ảo Android 12+, có thể cần vào **Cài đặt → Ứng dụng → MedMinder → Báo thức & lời nhắc** để bật thủ công nếu Play Protect chặn.
