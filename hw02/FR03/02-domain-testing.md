# FR-03 — Domain Testing

## 1. Mục tiêu

Áp dụng Domain Testing cho chức năng Quên mật khẩu và Đặt lại mật khẩu bằng cách:

1. Xác định biến đầu vào và biến trạng thái.
2. Chia mỗi biến thành các lớp tương đương hợp lệ và không hợp lệ.
3. Chọn một giá trị đại diện cho mỗi lớp.
4. Kết hợp các lớp để tạo test case, ưu tiên mỗi test âm chỉ vi phạm một điều kiện.

Boundary Value Analysis được trình bày trong tài liệu riêng và không được tính trùng vào số test case Domain Testing.

## 2. Xác định biến đầu vào

| Biến | Kiểu miền | Điều kiện theo đặc tả |
|---|---|---|
| Email | Chuỗi / trạng thái tài khoản | Đúng định dạng và đã đăng ký |
| OTP | Chuỗi ký tự | Đúng 6 chữ số |
| Quan hệ OTP–email | Quan hệ | OTP phải thuộc email đã yêu cầu |
| Trạng thái OTP | Trạng thái | Còn hạn và chưa được sử dụng |
| Mật khẩu mới | Chuỗi | Có chữ hoa, chữ thường, chữ số, ký tự đặc biệt cho phép và tối thiểu 8 ký tự |
| Xác nhận mật khẩu | Quan hệ | Giống hoàn toàn mật khẩu mới |

## 3. Bảng phân vùng tương đương

### 3.1. Email

| Mã lớp | Loại lớp | Mô tả lớp tương đương | Giá trị đại diện | Kết quả mong đợi |
|---|---|---|---|---|
| E-V1 | Hợp lệ | Đúng định dạng và đã đăng ký | `test@eshop.com` | Tạo OTP và chuyển sang bước 2 |
| E-I1 | Không hợp lệ | Đúng định dạng nhưng chưa đăng ký | `notfound@eshop.com` | Từ chối |
| E-I2 | Không hợp lệ | Sai định dạng email | `testeshop.com` | Từ chối |
| E-I3 | Không hợp lệ | Không cung cấp giá trị | Chuỗi rỗng | Từ chối |

### 3.2. OTP và trạng thái OTP

| Mã lớp | Loại lớp | Mô tả lớp tương đương | Giá trị đại diện | Kết quả mong đợi |
|---|---|---|---|---|
| O-V1 | Hợp lệ | OTP đúng định dạng, đúng mã, đúng email, còn hạn và chưa dùng | OTP hiện tại do SUT cấp | Cho phép kiểm tra mật khẩu mới |
| O-I1 | Không hợp lệ | Đủ 6 ký tự nhưng có ký tự không phải chữ số | `12A456` | Từ chối |
| O-I2 | Không hợp lệ | Không cung cấp OTP | Chuỗi rỗng | Từ chối |
| O-I3 | Không hợp lệ | Đúng 6 chữ số nhưng sai mã | Một mã khác OTP được cấp | Từ chối |
| O-I4 | Không hợp lệ | OTP do một email khác yêu cầu | OTP của tài khoản kiểm thử thứ hai | Từ chối |
| O-I5 | Không hợp lệ | OTP đã hết hạn | OTP quá thời hạn quy định | Từ chối |
| O-I6 | Không hợp lệ | OTP đã được sử dụng thành công | OTP vừa dùng để đổi mật khẩu | Từ chối tái sử dụng |

Độ dài 5, 6 và 7 chữ số được kiểm tra trong tài liệu BVA, nên không tạo lại test độ dài tại đây.

### 3.3. Thành phần mật khẩu mới

| Mã lớp | Loại lớp | Mô tả lớp tương đương | Giá trị đại diện | Kết quả mong đợi |
|---|---|---|---|---|
| P-V1 | Hợp lệ | Có đủ chữ hoa, chữ thường, chữ số và ký tự đặc biệt cho phép | `Newpass1!` | Chấp nhận khi các biến khác hợp lệ |
| P-I1 | Không hợp lệ | Thiếu chữ hoa | `newpass1!` | Từ chối |
| P-I2 | Không hợp lệ | Thiếu chữ thường | `NEWPASS1!` | Từ chối |
| P-I3 | Không hợp lệ | Thiếu chữ số | `Newpass!!` | Từ chối |
| P-I4 | Không hợp lệ | Thiếu ký tự đặc biệt | `Newpass12` | Từ chối |
| P-I5 | Không hợp lệ | Chỉ có ký tự đặc biệt ngoài danh sách cho phép | `Newpass1#` | Từ chối |
| P-I6 | Không hợp lệ | Không cung cấp mật khẩu mới | Chuỗi rỗng | Từ chối |

Biên độ dài 7, 8 và 9 ký tự được kiểm tra trong tài liệu BVA. Danh sách ký tự đặc biệt được đặc tả là `@`, `$`, `!`, `%`, `*`, `?`, `&`.

### 3.4. Xác nhận mật khẩu

| Mã lớp | Loại lớp | Mô tả lớp tương đương | Giá trị đại diện | Kết quả mong đợi |
|---|---|---|---|---|
| C-V1 | Hợp lệ | Giống hoàn toàn mật khẩu mới | `Newpass1!` / `Newpass1!` | Chấp nhận khi các biến khác hợp lệ |
| C-I1 | Không hợp lệ | Khác mật khẩu mới | `Newpass1!` / `Different1!` | Từ chối |
| C-I2 | Không hợp lệ | Không cung cấp xác nhận | `Newpass1!` / rỗng | Từ chối |

## 4. Base test và chiến lược kết hợp

Base test dùng một giá trị hợp lệ từ mỗi miền:

| Email | OTP | Trạng thái OTP | Mật khẩu mới | Xác nhận |
|---|---|---|---|---|
| `test@eshop.com` | OTP thực tế | Đúng email, còn hạn, chưa dùng | `Newpass1!` | `Newpass1!` |

Từ base test, mỗi test âm thay đúng một lớp hợp lệ bằng một lớp không hợp lệ. Cách này giúp xác định điều kiện nào gây ra kết quả từ chối.

## 5. Test case Domain Testing

### 5.1. Bước 1 — Yêu cầu OTP

| Test ID | Lớp bao phủ | Dữ liệu / bước thực hiện | Kết quả mong đợi | Kết quả thực tế | Trạng thái|
|---|---|---|---|---|---
| FR03-DT-01 | E-V1 | Nhập `test@eshop.com` và gửi yêu cầu. | Tạo và hiển thị OTP 6 chữ số; chuyển sang bước 2. | OTP gồm 4 chữ số | Fail |
| FR03-DT-02 | E-I1 | Nhập `notfound@eshop.com` và gửi yêu cầu. | Từ chối; không tạo OTP và không chuyển bước. | Từ chối | Pass |
| FR03-DT-03 | E-I2 | Nhập `testeshop.com` và gửi yêu cầu. | Từ chối do sai định dạng email. | Từ chối do không tìm được user | Fail |
| FR03-DT-04 | E-I3 | Để trống Email và gửi yêu cầu. | Từ chối do thiếu dữ liệu bắt buộc. | Bắt buộc điền email mới thực hiện được | Pass |

### 5.2. Bước 2 — OTP và trạng thái OTP

Tiền điều kiện: đang ở bước 2; nhập `Newpass1!` cho cả hai trường mật khẩu trừ khi test quy định khác.

| Test ID | Lớp bao phủ | Dữ liệu / bước thực hiện | Kết quả mong đợi | Kết quả thực tế | Trạng thái
|---|---|---|---|---|---
| FR03-DT-05 | O-V1 | Nhập OTP hiện tại của `test@eshop.com`. | OTP được chấp nhận và mật khẩu được đổi. | Thay đổi mật khẩu thất bại vì mật khẩu chưa đủ yêu cầu | Fail |
| FR03-DT-06 | O-I1 | Nhập `12A456`. | Từ chối; mật khẩu không đổi. | Không đổi được | Pass |
| FR03-DT-07 | O-I2 | Để trống OTP. | Từ chối do thiếu dữ liệu bắt buộc. | Không thể click nút | Pass | 
| FR03-DT-08 | O-I3 | Nhập một mã 6 chữ số khác OTP được cấp. | Từ chối OTP sai; mật khẩu không đổi. | Vẫn nhận otp  | Fail |
| FR03-DT-09 | O-I4 | Dùng OTP của tài khoản kiểm thử thứ hai cho `test@eshop.com`. | Từ chối vì OTP không thuộc email; mật khẩu không đổi. | Vẫn nhận otp nhưng mật khẩu không đạt | Fail |
| FR03-DT-10 | O-I6 | Đổi mật khẩu thành công rồi gửi lại chính OTP đó với mật khẩu hợp lệ khác. | Lần thứ hai bị từ chối. | Chức năng đổi mật khẩu không hoạt động do mật khẩu không được chấp nhận với bất kì định dạng nào | BLOCKED |

### 5.3. Bước 2 — Thành phần mật khẩu

Tiền điều kiện: dùng OTP hợp lệ, đúng email, còn hạn và chưa sử dụng. Xác nhận giống mật khẩu mới.

| Test ID | Lớp bao phủ | Mật khẩu mới | Kết quả mong đợi | Kết quả thực tế | Trạng thái 
|---|---|---|---|---|---
| FR03-DT-12 | P-V1 | `Newpass1!` | Chấp nhận và đổi mật khẩu. | Không đổi được mật khẩu | Fail
| FR03-DT-13 | P-I1 | `newpass1!` | Từ chối vì thiếu chữ hoa. | Từ chối vì định dạng chung | Faiil
| FR03-DT-14 | P-I2 | `NEWPASS1!` | Từ chối vì thiếu chữ thường. |  Từ chối vì định dạng chung | Fail
| FR03-DT-15 | P-I3 | `Newpass!!` | Từ chối vì thiếu chữ số. |  Từ chối vì định dạng chung | Fail
| FR03-DT-16 | P-I4 | `Newpass12` | Từ chối vì thiếu ký tự đặc biệt. |  Từ chối vì định dạng chung | Fail
| FR03-DT-17 | P-I5 | `Newpass1#` | Từ chối vì `#` không thuộc danh sách cho phép. |  Từ chối vì định dạng chung | Fail
| FR03-DT-18 | P-I6 | Để trống mật khẩu mới. | Từ chối do thiếu dữ liệu bắt buộc. | Bắt điền mật khẩu | Pass

### 5.4. Bước 2 — Xác nhận mật khẩu

Tiền điều kiện: dùng OTP hợp lệ và mật khẩu mới `Newpass1!`.

| Test ID | Lớp bao phủ | Xác nhận mật khẩu | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR03-DT-19 | C-V1 | `Newpass1!` | Chấp nhận khi các điều kiện khác hợp lệ. | Báo mật khẩu không đạt | Fail | — |
| FR03-DT-20 | C-I1 | `Different1!` | Từ chối vì hai mật khẩu không khớp. | UI chỉ có 1 ô mật khẩu mới | Fail | — |
| FR03-DT-21 | C-I2 | Để trống xác nhận. | Từ chối do thiếu dữ liệu bắt buộc. | Bắt buộc điền mật khẩu | Pass | — |
