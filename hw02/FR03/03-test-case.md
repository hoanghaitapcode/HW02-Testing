# FR-03 — Test case Quên mật khẩu và Đặt lại mật khẩu

## 1. Quy ước

- **Đối tượng:** Web EShop.
- **Tài khoản chuẩn:** `test@eshop.com`.
- **Mật khẩu ban đầu:** `Test1234!` nếu dữ liệu seed chưa bị thay đổi.
- **Mật khẩu hợp lệ mẫu:** `Newpass1!`.
- **OTP hợp lệ:** mã đang còn hiệu lực do hệ thống hiển thị sau khi gửi email.
- Các test làm thay đổi mật khẩu chỉ được chạy trên cơ sở dữ liệu kiểm thử riêng.
- `KQ thực tế`, `Trạng thái` và `Ảnh` được cập nhật sau khi thực thi.

Trạng thái hợp lệ: `Chưa thực hiện`, `Pass`, `Fail`, `Blocked`.

## 2. Test giao diện và điều hướng

| ID | Kỹ thuật / yêu cầu | Các bước và dữ liệu | Kết quả mong đợi | KQ thực tế | Trạng thái | Ảnh |
|---|---|---|---|---|---|---|
| FR03-TC01 | FR03-R04, R16, R18 | Mở trang Quên mật khẩu; quan sát form bước 1 và kiểm tra trường Email bằng DevTools nếu cần. | Có Step Indicator bước 1/2; trường Email dùng `type="email"`; nhãn trường bắt buộc có `*`. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC02 | FR03-R05 | Tại bước 1, chọn **Quay lại đăng nhập**. | Điều hướng về trang Đăng nhập. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC03 | FR03-R03, R04, R06, R17, R18 | Nhập `test@eshop.com`, gửi yêu cầu OTP; quan sát form bước 2 và kiểm tra kiểu trường. | OTP được hiển thị trong môi trường demo; có Step Indicator bước 2/2; có đủ OTP, mật khẩu mới, xác nhận mật khẩu; hai trường mật khẩu dùng `type="password"`; các trường bắt buộc có `*`. | Chưa ghi | Chưa thực hiện | — |

## 3. Test email ở bước 1

| ID | Lớp miền | Các bước và dữ liệu | Kết quả mong đợi | KQ thực tế | Trạng thái | Ảnh |
|---|---|---|---|---|---|---|
| FR03-TC04 | E01 | Nhập email đã đăng ký `test@eshop.com`; gửi yêu cầu. | Chấp nhận; sinh và hiển thị OTP đúng 6 chữ số; chuyển sang bước 2. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC05 | E02 | Nhập `notfound@eshop.com`; gửi yêu cầu. | Từ chối và hiển thị thông báo lỗi; không chuyển sang bước 2. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC06 | E03 | Nhập `testeshop.com`; gửi yêu cầu. | HTML5 hoặc ứng dụng từ chối vì sai định dạng; không tạo OTP. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC07 | E04 | Để trống Email; gửi yêu cầu. | Từ chối vì trường bắt buộc; không tạo OTP. | Chưa ghi | Chưa thực hiện | — |

## 4. Test định dạng và trạng thái OTP

Tiền điều kiện chung: đã yêu cầu OTP cho `test@eshop.com` và đang ở bước 2. Dùng `Newpass1!` cho cả hai trường mật khẩu trừ khi test quy định khác.

| ID | Lớp miền / BVA | Dữ liệu OTP và thao tác | Kết quả mong đợi | KQ thực tế | Trạng thái | Ảnh |
|---|---|---|---|---|---|---|
| FR03-TC08 | O02; BVA 5 | Nhập `12345`; gửi form. | Từ chối OTP có 5 chữ số; mật khẩu không đổi. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC09 | O01; BVA 6 | Nhập OTP 6 chữ số thực tế do hệ thống cấp; gửi form. | Chấp nhận định dạng OTP và đặt lại mật khẩu thành công. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC10 | O03; BVA 7 | Nhập `1234567`; gửi form. | Từ chối OTP có 7 chữ số; mật khẩu không đổi. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC11 | O04 | Nhập `12A456`; gửi form. | Từ chối OTP chứa ký tự không phải chữ số; mật khẩu không đổi. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC12 | O05 | Để trống OTP; gửi form. | Từ chối vì OTP bắt buộc; mật khẩu không đổi. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC13 | S02 | Nhập một mã 6 chữ số khác OTP được cấp; gửi form. | Từ chối OTP sai; mật khẩu không đổi. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC14 | S03; FR03-R13 | Yêu cầu OTP cho hai tài khoản kiểm thử; dùng OTP của tài khoản thứ hai để đặt lại mật khẩu cho `test@eshop.com`. | Từ chối vì OTP không thuộc email đang đặt lại; mật khẩu không đổi. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC15 | S04; FR03-R14 | Dùng OTP sau khi thời hạn được đặc tả đã hết. | Từ chối OTP hết hạn; mật khẩu không đổi. Nếu SUT không công bố thời hạn, ghi `Blocked` và nguyên nhân. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC16 | S05; FR03-R15 | Đặt lại mật khẩu thành công, sau đó gửi lại chính OTP đó với một mật khẩu hợp lệ khác. | Lần thứ hai bị từ chối; OTP đã dùng không thể tái sử dụng. | Chưa ghi | Chưa thực hiện | — |

## 5. Test mật khẩu mới

Tiền điều kiện chung: đang ở bước 2 và dùng OTP hợp lệ, còn hạn, thuộc đúng email. Xác nhận mật khẩu giống mật khẩu mới trừ khi test quy định khác.

| ID | Lớp miền / BVA | Mật khẩu mới | Kết quả mong đợi | KQ thực tế | Trạng thái | Ảnh |
|---|---|---|---|---|---|---|
| FR03-TC17 | P02; BVA 7 | `Newp1!a` (7 ký tự) | Từ chối vì ngắn hơn 8 ký tự. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC18 | P01; BVA 8 | `Newpa1!x` (8 ký tự) | Chấp nhận và đặt lại mật khẩu thành công. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC19 | P01; BVA 9 | `Newpass1!` (9 ký tự) | Chấp nhận và đặt lại mật khẩu thành công. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC20 | P03 | `newpass1!` | Từ chối vì thiếu chữ hoa. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC21 | P04 | `NEWPASS1!` | Từ chối vì thiếu chữ thường. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC22 | P05 | `Newpass!!` | Từ chối vì thiếu chữ số. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC23 | P06 | `Newpass12` | Từ chối vì thiếu ký tự đặc biệt. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC24 | P07 | `Newpass1#` | Từ chối vì `#` không thuộc danh sách ký tự đặc biệt được quy định. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC25 | P08 | Để trống mật khẩu mới và xác nhận. | Từ chối vì trường bắt buộc. | Chưa ghi | Chưa thực hiện | — |

## 6. Test xác nhận mật khẩu và kết quả sau đặt lại

| ID | Lớp miền / yêu cầu | Các bước và dữ liệu | Kết quả mong đợi | KQ thực tế | Trạng thái | Ảnh |
|---|---|---|---|---|---|---|
| FR03-TC26 | C02; FR03-R12 | Mật khẩu mới `Newpass1!`; xác nhận `Different1!`; dùng OTP hợp lệ và gửi form. | Từ chối vì hai mật khẩu không khớp; mật khẩu không đổi. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC27 | C03; FR03-R12 | Mật khẩu mới `Newpass1!`; để trống xác nhận; dùng OTP hợp lệ và gửi form. | Từ chối vì trường xác nhận bắt buộc; mật khẩu không đổi. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC28 | K01 | Đặt lại thành công bằng OTP hợp lệ và `Newpass1!`; quay lại trang đăng nhập; lần lượt thử mật khẩu cũ và mới. | Mật khẩu cũ không đăng nhập được; mật khẩu mới đăng nhập thành công. | Chưa ghi | Chưa thực hiện | — |
| FR03-TC29 | FR03-R19 | Gửi một bộ dữ liệu không hợp lệ làm ứng dụng hiển thị lỗi; quan sát vị trí lỗi. | Thông báo lỗi xuất hiện phía trên nút gửi form. | Chưa ghi | Chưa thực hiện | — |

## 7. Kiểm tra tính ngẫu nhiên của OTP

| ID | Yêu cầu | Các bước và dữ liệu | Kết quả mong đợi | KQ thực tế | Trạng thái | Ảnh |
|---|---|---|---|---|---|---|
| FR03-TC30 | FR03-R02 | Yêu cầu OTP nhiều lần trong các phiên kiểm thử độc lập; ghi lại 10 mã được sinh. | Mọi mã đều có đúng 6 chữ số và tập mẫu có nhiều hơn một giá trị. Kết quả này chỉ phát hiện lỗi rõ ràng, không chứng minh đầy đủ tính ngẫu nhiên. | Chưa ghi | Chưa thực hiện | — |

## 8. Tổng hợp thiết kế

| Nhóm | Số test case |
|---|---:|
| Giao diện và điều hướng | 3 |
| Email bước 1 | 4 |
| Định dạng và trạng thái OTP | 9 |
| Mật khẩu mới | 9 |
| Xác nhận và kết quả đặt lại | 4 |
| Kiểm tra mẫu OTP | 1 |
| **Tổng cộng** | **30** |

Lưu ý: FR03-TC09, FR03-TC18, FR03-TC19, FR03-TC28 và FR03-TC30 có thể làm thay đổi OTP hoặc mật khẩu. Khi thực thi phải tạo OTP mới theo đúng thứ tự hướng dẫn và ghi lại mật khẩu hiện tại của dữ liệu kiểm thử.
