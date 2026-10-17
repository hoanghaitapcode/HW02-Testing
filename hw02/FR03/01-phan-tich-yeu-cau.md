# FR-03 — Phân tích yêu cầu

## 1. Chức năng được kiểm thử

- **Mã chức năng:** FR-03
- **Tên chức năng:** Quên mật khẩu và đặt lại mật khẩu trên Web
- **Mục tiêu:** Kiểm tra quy trình hai bước lấy OTP và đặt lại mật khẩu.
- **Nguồn yêu cầu:** `README.md` của EShop, gồm FR-01, FR-03, FR-22 và SEC-07.

## 2. Các yêu cầu có thể kiểm thử

| ID | Yêu cầu |
|---|---|
| FR03-R01 | Người dùng nhập email đã đăng ký để yêu cầu OTP. |
| FR03-R02 | Hệ thống tạo OTP ngẫu nhiên gồm đúng 6 chữ số. |
| FR03-R03 | Trong môi trường demo, OTP được hiển thị trực tiếp trên màn hình. |
| FR03-R04 | Quy trình gồm hai bước và giao diện phải có chỉ báo bước rõ ràng. |
| FR03-R05 | Ở bước 1 phải có cách quay lại trang đăng nhập. |
| FR03-R06 | Bước 2 phải có ba trường: OTP, mật khẩu mới và xác nhận mật khẩu mới. |
| FR03-R07 | Mật khẩu mới có tối thiểu 8 ký tự. |
| FR03-R08 | Mật khẩu mới có ít nhất một chữ hoa. |
| FR03-R09 | Mật khẩu mới có ít nhất một chữ thường. |
| FR03-R10 | Mật khẩu mới có ít nhất một chữ số. |
| FR03-R11 | Mật khẩu mới có ít nhất một ký tự đặc biệt thuộc `@`, `$`, `!`, `%`, `*`, `?`, `&`. |
| FR03-R12 | Mật khẩu mới và xác nhận mật khẩu phải giống nhau. |
| FR03-R13 | OTP chỉ hợp lệ với email đã yêu cầu OTP đó. |
| FR03-R14 | OTP phải có thời hạn sử dụng. |
| FR03-R15 | OTP phải bị vô hiệu hóa sau khi đã sử dụng thành công. |
| FR03-R16 | Trường email phải dùng kiểu `email`. |
| FR03-R17 | Các trường mật khẩu phải dùng kiểu `password`. |
| FR03-R18 | Trường bắt buộc phải có ký hiệu `*` bên cạnh nhãn. |
| FR03-R19 | Thông báo lỗi phải nằm phía trên nút gửi biểu mẫu. |

## 3. Tiền điều kiện và dữ liệu chuẩn

- EShop đang chạy trong môi trường kiểm thử.
- Cơ sở dữ liệu có tài khoản `test@eshop.com`.
- Mật khẩu ban đầu của tài khoản thử nghiệm là `Test1234!` nếu dữ liệu seed chưa bị thay đổi.
- Người kiểm thử đang ở trang Quên mật khẩu và chưa đăng nhập.
- Không dùng cơ sở dữ liệu đang làm việc để chạy test phá hủy; cần dùng bản sao kiểm thử riêng.

## 4. Kết quả chính cần quan sát

- Nội dung và thuộc tính của các trường trên giao diện.
- Việc chuyển từ bước 1 sang bước 2.
- Độ dài và kiểu ký tự của OTP hiển thị.
- Thông báo chấp nhận hoặc từ chối dữ liệu.
- Khả năng đăng nhập bằng mật khẩu mới sau khi đặt lại thành công.
- Khả năng tái sử dụng OTP sau khi đã dùng.

## 5. Điểm chưa được đặc tả rõ

Các mục sau không được tự đặt thành tiêu chí Pass/Fail nếu chưa có yêu cầu bổ sung:

- Thời lượng hết hạn cụ thể của OTP.
- Độ dài tối đa của email, OTP hoặc mật khẩu.
- Số lần được nhập sai OTP.
- Cách xử lý khoảng trắng đầu/cuối.
- Nội dung chính xác của từng thông báo lỗi.
- Có cho phép yêu cầu OTP nhiều lần và OTP cũ có bị hủy ngay hay không.

## 6. Phạm vi bước tiếp theo

Bước tiếp theo sẽ xác định lớp tương đương, giá trị biên và điều kiện kết hợp. Chưa ghi Pass/Fail ở giai đoạn phân tích yêu cầu.
