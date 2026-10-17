# FR-04 — Boundary Value Analysis (BVA)

## 1. Biên được kiểm thử

Đặc tả FR-04 yêu cầu số điện thoại:

- Bắt đầu bằng chữ số `0`.
- Có từ 10 đến 11 chữ số, tính cả chữ số `0` đầu tiên.

Độ dài hợp lệ là miền đóng `[10, 11]`. Theo BVA, chọn các điểm ngay ngoài và tại hai biên:

| Điểm BVA | Độ dài | Giá trị mẫu | Kết quả kỳ vọng |
|---|---:|---|---|
| Ngay dưới biên dưới | 9 | `091234567` | Không hợp lệ |
| Tại biên dưới | 10 | `0912345678` | Hợp lệ |
| Tại biên trên | 11 | `09123456789` | Hợp lệ |
| Ngay trên biên trên | 12 | `091234567890` | Không hợp lệ |

Tất cả giá trị đều bắt đầu bằng `0` và chỉ gồm chữ số để cô lập riêng biên độ dài. Điều kiện bắt đầu bằng `0`, ký tự không phải số và khoảng trắng đã được kiểm tra trong Domain Testing.

## 2. Tiền điều kiện

- User đã đăng nhập bằng JWT hợp lệ.
- Hồ sơ ban đầu đã được ghi lại.
- Có thể thực hiện trên Mobile UI hoặc API `/api/users/me`.
- Khi test API, giữ nguyên Họ Tên và Địa chỉ hợp lệ để chỉ thay đổi số điện thoại.

## 3. Test case BVA

| Test ID | Điểm biên | Dữ liệu và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR04-BVA-PHONE-09 | Ngay dưới biên dưới | Cập nhật `phone=091234567` rồi lưu. | Từ chối vì chỉ có 9 chữ số; số điện thoại cũ không đổi. | Cập nhật và hiển thị được | Fail |
| FR04-BVA-PHONE-10 | Tại biên dưới | Cập nhật `phone=0912345678` rồi lưu. | Chấp nhận vì có đúng 10 chữ số và bắt đầu bằng `0`. |  Cập nhật và hiển thị được | Pass|
| FR04-BVA-PHONE-11 | Tại biên trên | Cập nhật `phone=09123456789` rồi lưu. | Chấp nhận vì có đúng 11 chữ số và bắt đầu bằng `0`. |  Cập nhật và hiển thị được | Fail |
| FR04-BVA-PHONE-12 | Ngay trên biên trên | Cập nhật `phone=091234567890` rồi lưu. | Từ chối vì có 12 chữ số; số điện thoại cũ không đổi. |  Cập nhật và hiển thị được | Fail |

