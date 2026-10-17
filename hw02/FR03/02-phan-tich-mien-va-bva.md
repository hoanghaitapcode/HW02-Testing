# FR-03 — Phân tích miền và giá trị biên

## 1. Phạm vi phân tích

Tài liệu này xây dựng các lớp tương đương và giá trị biên cho chức năng Quên mật khẩu và Đặt lại mật khẩu. Kết quả Pass/Fail chỉ được ghi sau khi thực thi test.

Các biến đầu vào chính:

1. Email yêu cầu OTP.
2. Mã OTP.
3. Mật khẩu mới.
4. Xác nhận mật khẩu mới.
5. Trạng thái của OTP và quan hệ giữa OTP với email.

## 2. Phân vùng tương đương

### 2.1. Email yêu cầu OTP

| Lớp | Loại | Mô tả | Dữ liệu đại diện | Kết quả mong đợi |
|---|---|---|---|---|
| E01 | Hợp lệ | Email đúng định dạng và đã đăng ký | `test@eshop.com` | Chấp nhận yêu cầu, tạo OTP và chuyển sang bước 2 |
| E02 | Không hợp lệ | Email đúng định dạng nhưng chưa đăng ký | `notfound@eshop.com` | Từ chối yêu cầu và hiển thị lỗi |
| E03 | Không hợp lệ | Email sai định dạng | `testeshop.com` | Không gửi biểu mẫu hoặc hiển thị lỗi định dạng |
| E04 | Không hợp lệ | Email rỗng | Chuỗi rỗng | Không gửi biểu mẫu hoặc hiển thị lỗi bắt buộc |

### 2.2. Định dạng OTP

| Lớp | Loại | Mô tả | Dữ liệu đại diện | Kết quả mong đợi |
|---|---|---|---|---|
| O01 | Hợp lệ về định dạng | Đúng 6 ký tự và tất cả đều là chữ số | OTP 6 chữ số do hệ thống cấp | Tiếp tục kiểm tra trạng thái và quan hệ OTP–email |
| O02 | Không hợp lệ | Ít hơn 6 chữ số | `12345` | Từ chối |
| O03 | Không hợp lệ | Nhiều hơn 6 chữ số | `1234567` | Từ chối |
| O04 | Không hợp lệ | Có ký tự không phải chữ số | `12A456` | Từ chối |
| O05 | Không hợp lệ | Rỗng | Chuỗi rỗng | Từ chối |

Một OTP như `012345` vẫn có đúng 6 chữ số. Không được chuyển OTP sang số nguyên làm mất số `0` ở đầu.

### 2.3. Trạng thái và quyền sử dụng OTP

| Lớp | Loại | Mô tả | Kết quả mong đợi |
|---|---|---|---|
| S01 | Hợp lệ | OTP đúng, còn hạn, chưa dùng và thuộc email đang đặt lại | Cho phép đặt lại mật khẩu nếu mật khẩu hợp lệ |
| S02 | Không hợp lệ | OTP 6 chữ số nhưng không đúng mã đã cấp | Từ chối, mật khẩu không thay đổi |
| S03 | Không hợp lệ | OTP thuộc một email khác | Từ chối, mật khẩu không thay đổi |
| S04 | Không hợp lệ | OTP đã hết hạn | Từ chối, mật khẩu không thay đổi |
| S05 | Không hợp lệ | OTP đã được dùng thành công trước đó | Từ chối tái sử dụng |

### 2.4. Mật khẩu mới

| Lớp | Loại | Mô tả | Dữ liệu đại diện | Kết quả mong đợi |
|---|---|---|---|---|
| P01 | Hợp lệ | Từ 8 ký tự, có chữ hoa, chữ thường, chữ số và ký tự đặc biệt được cho phép | `Newpass1!` | Chấp nhận nếu các đầu vào khác hợp lệ |
| P02 | Không hợp lệ | Ít hơn 8 ký tự | `Newp1!a` | Từ chối |
| P03 | Không hợp lệ | Thiếu chữ hoa | `newpass1!` | Từ chối |
| P04 | Không hợp lệ | Thiếu chữ thường | `NEWPASS1!` | Từ chối |
| P05 | Không hợp lệ | Thiếu chữ số | `Newpass!!` | Từ chối |
| P06 | Không hợp lệ | Thiếu ký tự đặc biệt | `Newpass12` | Từ chối |
| P07 | Không hợp lệ | Chỉ dùng ký tự đặc biệt ngoài danh sách cho phép | `Newpass1#` | Từ chối |
| P08 | Không hợp lệ | Rỗng | Chuỗi rỗng | Từ chối |

Danh sách ký tự đặc biệt được yêu cầu cho phép: `@`, `$`, `!`, `%`, `*`, `?`, `&`.

### 2.5. Xác nhận mật khẩu

| Lớp | Loại | Mô tả | Kết quả mong đợi |
|---|---|---|---|
| C01 | Hợp lệ | Xác nhận giống hoàn toàn mật khẩu mới | Chấp nhận nếu các đầu vào khác hợp lệ |
| C02 | Không hợp lệ | Xác nhận khác mật khẩu mới | Từ chối |
| C03 | Không hợp lệ | Xác nhận rỗng | Từ chối |

## 3. Phân tích giá trị biên (BVA)

### 3.1. Biên độ dài OTP

Yêu cầu chính xác 6 chữ số, vì vậy miền hợp lệ chỉ có độ dài 6.

| Điểm biên | Độ dài | Dữ liệu đại diện | Hợp lệ theo đặc tả |
|---|---:|---|---|
| Ngay dưới biên | 5 | `12345` | Không |
| Tại biên | 6 | OTP thực tế do hệ thống cấp | Có |
| Ngay trên biên | 7 | `1234567` | Không |

### 3.2. Biên độ dài tối thiểu của mật khẩu

Yêu cầu tối thiểu 8 ký tự và không quy định độ dài tối đa.

| Điểm biên | Độ dài | Dữ liệu đại diện | Hợp lệ theo đặc tả |
|---|---:|---|---|
| Ngay dưới biên | 7 | `Newp1!a` | Không |
| Tại biên | 8 | `Newpa1!x` | Có |
| Ngay trên biên | 9 | `Newpass1!` | Có |

Mỗi dữ liệu mật khẩu dùng cho BVA vẫn phải thỏa các quy tắc thành phần, ngoại trừ điều kiện độ dài đang được cô lập.

### 3.3. Các yêu cầu không áp dụng BVA số

- Email đã đăng ký/chưa đăng ký là miền phân loại.
- Các điều kiện có/không có chữ hoa, chữ thường, chữ số, ký tự đặc biệt là miền Boolean.
- Quan hệ OTP đúng email/sai email, còn hạn/hết hạn, đã dùng/chưa dùng là miền trạng thái.
- Đặc tả không cho biết thời lượng hết hạn OTP và độ dài tối đa của mật khẩu, nên không tạo giá trị biên giả định cho hai mục này.

## 4. Tổ hợp điều kiện quan trọng

| Tổ hợp | Email | OTP | Mật khẩu | Xác nhận | Kết quả mong đợi |
|---|---|---|---|---|---|
| K01 | Đã đăng ký | Đúng, còn hạn, cùng email | Hợp lệ | Khớp | Đặt lại thành công |
| K02 | Đã đăng ký | Sai | Hợp lệ | Khớp | Từ chối |
| K03 | Đã đăng ký | Thuộc email khác | Hợp lệ | Khớp | Từ chối |
| K04 | Đã đăng ký | Đúng | Không hợp lệ | Khớp | Từ chối |
| K05 | Đã đăng ký | Đúng | Hợp lệ | Không khớp | Từ chối |
| K06 | Đã đăng ký | Đã dùng | Hợp lệ | Khớp | Từ chối tái sử dụng |

## 5. Quy tắc thiết kế test ở bước tiếp theo

- Mỗi test âm chỉ thay đổi một điều kiện chính khi có thể để xác định nguyên nhân lỗi.
- Dùng chung một test cho lớp tương đương và điểm biên nếu cùng dữ liệu, tránh đếm trùng.
- Test giao diện kiểm tra thuộc tính trường, ký hiệu bắt buộc, Step Indicator và vị trí thông báo lỗi.
- Test chức năng kiểm tra cả phản hồi trên màn hình và việc mật khẩu thực tế có thay đổi hay không.
- Trạng thái thực thi ban đầu của mọi test case phải là `Chưa thực hiện`.
