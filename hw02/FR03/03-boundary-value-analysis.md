# FR-03 — Boundary Value Analysis (BVA)

## 1. Phạm vi và nguyên tắc

Tài liệu này áp dụng riêng kỹ thuật Boundary Value Analysis cho các giới hạn số được đặc tả rõ trong FR-03:

- OTP phải có **đúng 6 chữ số**.
- Mật khẩu mới có **tối thiểu 8 ký tự**.

Với mỗi biên, chọn các điểm `n-1`, `n`, `n+1`. Các quy tắc thành phần của mật khẩu (chữ hoa, chữ thường, chữ số, ký tự đặc biệt) thuộc Domain Testing và không lặp lại trong bảng này.

## 2. Các biên được xác định

### 2.1. Độ dài OTP

Đặc tả chỉ cho phép độ dài chính xác 6 chữ số. Vì vậy:

| Điểm BVA | Độ dài | Miền kỳ vọng |
|---|---:|---|
| Ngay dưới biên | 5 | Không hợp lệ |
| Tại biên | 6 | Hợp lệ |
| Ngay trên biên | 7 | Không hợp lệ |

Một giá trị đủ 6 chữ số phải đồng thời là OTP thật được cấp cho đúng email; chỉ nhập một chuỗi 6 chữ số bất kỳ không chứng minh được hệ thống chấp nhận độ dài 6.

### 2.2. Độ dài mật khẩu mới

Đặc tả quy định tối thiểu 8 ký tự và không quy định độ dài tối đa. Các giá trị BVA là:

| Điểm BVA | Độ dài | Giá trị mẫu | Miền kỳ vọng |
|---|---:|---|---|
| Ngay dưới biên | 7 | `Newp1!a` | Không hợp lệ |
| Tại biên | 8 | `Newpa1!x` | Hợp lệ |
| Ngay trên biên | 9 | `Newpass1!` | Hợp lệ |

Ba giá trị đều có chữ hoa, chữ thường, chữ số và ký tự đặc biệt cho phép để cô lập riêng biên độ dài.

## 3. Tiền điều kiện dùng chung

- EShop đang chạy ở môi trường kiểm thử riêng.
- Có tài khoản `test@eshop.com`.
- Đã yêu cầu OTP cho đúng email.
- Với test mật khẩu, dùng OTP thực tế đang còn hạn và chưa sử dụng.
- Xác nhận mật khẩu bằng đúng mật khẩu mới, trừ khi một test ghi khác.
- Nếu SUT chỉ sinh OTP 4 chữ số, không gọi mã 4 chữ số là dữ liệu hợp lệ theo đặc tả 6 chữ số. Ghi nhận đây là lỗi riêng và đánh dấu test biên 6 chữ số là `Blocked` nếu không thể tạo dữ liệu kiểm thử có kiểm soát.

## 4. Test case BVA độ dài OTP

| Test ID | Biên | Dữ liệu / bước thực hiện | Kết quả mong đợi | Kết quả thực tế | Trạng thái | Bằng chứng |
|---|---|---|---|---|---|---|
| FR03-BVA-OTP-05 | `n-1` | Ở bước 2, nhập OTP `12345`, mật khẩu mới hợp lệ và gửi form. | Từ chối vì OTP có 5 chữ số; mật khẩu không đổi. | Mật khẩu không đạt | Blocked | — |
| FR03-BVA-OTP-06 | `n` | Dùng một OTP **thực tế gồm 6 chữ số** do hệ thống cấp, nhập cùng mật khẩu hợp lệ và gửi form. | Chấp nhận OTP về độ dài và cho phép đặt lại mật khẩu. Nếu SUT không thể cấp OTP 6 chữ số, ghi `Blocked` với nguyên nhân; không đánh dấu Pass bằng OTP 4 chữ số. | OTP gửi về chỉ 4 chữ số | BLOCKED | — |
| FR03-BVA-OTP-07 | `n+1` | Ở bước 2, nhập OTP `1234567`, mật khẩu mới hợp lệ và gửi form. | Từ chối vì OTP có 7 chữ số; mật khẩu không đổi. | OTP Chi gửi 4 số | Blocked | — |

## 5. Test case BVA độ dài mật khẩu

Các test dưới đây dùng OTP mà SUT thực tế cấp để cô lập riêng biên độ dài mật khẩu. Nếu OTP không đúng 6 chữ số, ghi chú rõ lỗi OTP trong bằng chứng; không sửa kết quả độ dài mật khẩu thành Pass chỉ vì đã dùng dữ liệu không đúng đặc tả.

| Test ID | Biên | Mật khẩu mới và xác nhận | Kết quả mong đợi | Kết quả thực tế | Trạng thái | Bằng chứng |
|---|---|---|---|---|---|---|
| FR03-BVA-PWD-07 | `n-1` | Mới: `Newp1!a`; xác nhận: `Newp1!a` | Từ chối vì mật khẩu có 7 ký tự. | Từ chối với lý do chung là mật khẩu yếu | blocked | — |
| FR03-BVA-PWD-08 | `n` | Mới: `Newpa1!x`; xác nhận: `Newpa1!x` | Chấp nhận vì mật khẩu có đúng 8 ký tự và đủ thành phần. | Từ chối với lý do chung là mật khẩu yếu | blocked | — |
| FR03-BVA-PWD-09 | `n+1` | Mới: `Newpass1!`; xác nhận: `Newpass1!` | Chấp nhận vì mật khẩu có 9 ký tự và đủ thành phần. | Từ chối với lý do chung là mật khẩu yếu | blocked | — |

## 6. Không áp dụng BVA số

Không tạo thêm BVA cho các mục sau vì đặc tả chưa cung cấp giới hạn số:

- Độ dài tối đa của email.
- Độ dài tối đa của mật khẩu.
- Thời lượng cụ thể để OTP hết hạn.
- Số lần nhập sai OTP.
- Số lần yêu cầu lại OTP.

Các mục này được ghi nhận trong Domain Testing hoặc đánh dấu `Blocked` khi thiếu điều kiện đặc tả, không tự tạo biên giả định.

## 7. Tổng hợp BVA

| Nhóm | Số test case |
|---|---:|
| BVA OTP | 3 |
| BVA mật khẩu | 3 |
| **Tổng số test case BVA** | **6** |
| Đã thực hiện | 0 |
| Pass | 0 |
| Fail | 0 |
| Blocked | 0 |
| Chưa thực hiện | 6 |
