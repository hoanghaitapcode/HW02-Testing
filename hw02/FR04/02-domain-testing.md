# FR-04 — Domain Testing quản lý hồ sơ cá nhân Mobile

## 1. Phạm vi

FR-04 kiểm tra người dùng đã đăng nhập có thể cập nhật hồ sơ của chính mình trên Mobile, đồng thời không thể sửa Email hoặc thuộc tính `role`.

Các trường được kiểm tra:

- Họ Tên: `name`.
- Số điện thoại: `phone`.
- Địa chỉ giao hàng mặc định: `shipping_address`.
- Email và `role`: các thuộc tính bị giới hạn.

Không thêm cột `Bằng chứng`; ảnh lỗi được quản lý riêng trong bug report nếu phát hiện lỗi.

## 2. Biến miền và lớp tương đương

### 2.1. Xác thực và sở hữu

| Mã lớp | Điều kiện | Kết quả kỳ vọng |
|---|---|---|
| A1 | Có JWT hợp lệ của chính người dùng | Được đọc và cập nhật hồ sơ của mình |
| A2 | Không có JWT | Từ chối API hồ sơ |
| A3 | JWT sai hoặc hết hạn | Từ chối API hồ sơ |
| A4 | Cố truy cập hồ sơ người dùng khác | Từ chối hoặc không trả dữ liệu người khác |

### 2.2. Họ Tên và Địa chỉ

| Mã lớp | Dữ liệu | Kết quả kỳ vọng |
|---|---|---|
| N1 | Họ Tên mới thông thường | Lưu đúng giá trị mới |
| N2 | Họ Tên rỗng hoặc chỉ khoảng trắng | Chưa đặc tả bắt buộc; ghi nhận hành vi |
| N3 | Địa chỉ mới thông thường | Lưu đúng giá trị mới |
| N4 | Địa chỉ rỗng hoặc chỉ khoảng trắng | Chưa đặc tả bắt buộc; ghi nhận hành vi |

### 2.3. Số điện thoại

| Mã lớp | Dữ liệu | Kết quả kỳ vọng |
|---|---|---|
| P1 | Bắt đầu bằng `0`, có 10 hoặc 11 chữ số | Hợp lệ |
| P2 | Không bắt đầu bằng `0` | Không hợp lệ |
| P3 | Có chữ cái, khoảng trắng hoặc ký tự đặc biệt | Không hợp lệ |
| P4 | Không cung cấp số điện thoại | Chưa đặc tả rõ; ghi nhận hành vi |

Độ dài 9, 10, 11 và 12 chữ số được kiểm tra riêng bằng BVA, không lặp lại trong Domain Testing.

### 2.4. Thuộc tính bị giới hạn

| Mã lớp | Thao tác | Kết quả kỳ vọng |
|---|---|---|
| R1 | Không gửi Email/role khi cập nhật | Các thuộc tính bị giới hạn vẫn giữ nguyên |
| R2 | Gửi Email mới từ client | UI không cho sửa hoặc API từ chối/bỏ qua thay đổi |
| R3 | Gửi `role=admin` từ client | Không được tự nâng quyền; role vẫn giữ nguyên |

## 3. Test đọc và cập nhật hồ sơ chính mình

| Test ID | Lớp bao phủ | Điều kiện và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR04-DT-01 | A1 | Đăng nhập User và mở màn hình Hồ sơ trên Mobile. | Hiển thị đúng hồ sơ của User đang đăng nhập. | Hiển thị đúng | Pass |
| FR04-DT-02 | A1, N1 | Đổi Họ Tên thành `Nguyen Van Test` và lưu. | Lưu thành công; tải lại vẫn hiển thị tên mới. | Đổi và tải lại hiển thị thành công | Pass |
| FR04-DT-03 | A1, N3 | Đổi địa chỉ thành `12 Nguyen Hue, Quan 1` và lưu. | Lưu thành công; tải lại vẫn hiển thị địa chỉ mới. |  Đổi và tải lại hiển thị thành công | Pass |
| FR04-DT-04 | A1, P1 | Đổi số điện thoại thành `0912345678` và lưu. | Lưu thành công vì bắt đầu bằng 0 và thuộc miền hợp lệ. |  Đổi và tải lại hiển thị thành công | Pass  |
| FR04-DT-05 | A1, P1 | Đổi số điện thoại thành `09123456789` và lưu. | Lưu thành công vì có 11 chữ số hợp lệ. |  Đổi và tải lại hiển thị thành công | Pass  |
| FR04-DT-06 | A1, N2 | Xóa Họ Tên hoặc nhập chỉ khoảng trắng rồi lưu. | Ghi nhận hành vi; nếu đặc tả không quy định bắt buộc thì đánh dấu `Ambiguous`, không tự kết luận Fail. | Thành công |Ambiguous |
| FR04-DT-07 | A1, N4 | Xóa địa chỉ hoặc nhập chỉ khoảng trắng rồi lưu. | Ghi nhận hành vi; nếu đặc tả không quy định bắt buộc thì đánh dấu `Ambiguous`, không tự kết luận Fail. | Thành công |Ambiguous |

## 4. Test miền số điện thoại ngoài biên phân loại

| Test ID | Lớp bao phủ | Dữ liệu và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR04-DT-08 | P2 | Nhập `1912345678` và lưu. | Từ chối vì không bắt đầu bằng `0`. | Chưa ghi | Chưa thực hiện |
| FR04-DT-09 | P3 | Nhập `09A2345678` và lưu. | Từ chối vì có chữ cái. | Chưa ghi | Chưa thực hiện |
| FR04-DT-10 | P3 | Nhập `09 2345678` và lưu. | Từ chối vì có khoảng trắng. | Chưa ghi | Chưa thực hiện |
| FR04-DT-11 | P3 | Nhập `09-234-5678` và lưu. | Từ chối vì có dấu gạch không thuộc dạng chữ số thuần. | Chưa ghi | Chưa thực hiện |
| FR04-DT-12 | P4 | Xóa số điện thoại rồi lưu. | Ghi nhận hành vi theo đặc tả thực tế; không tự coi là lỗi nếu tính bắt buộc chưa được quy định. | Chưa ghi | Chưa thực hiện |

## 5. Test Email và role bị giới hạn

| Test ID | Lớp bao phủ | Điều kiện và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR04-DT-13 | R1 | Cập nhật name, phone, shipping_address mà không gửi Email/role. | Ba trường được cập nhật; Email và role giữ nguyên. | Chưa ghi | Chưa thực hiện |
| FR04-DT-14 | R2 | Trên Mobile UI, tìm cách sửa Email hoặc nhập Email mới nếu có trường này. | Email không có trường chỉnh sửa hoặc không thay đổi sau khi lưu. | Chưa ghi | Chưa thực hiện |
| FR04-DT-15 | R2 | Gửi request cập nhật kèm Email mới. | API từ chối hoặc bỏ qua Email; Email cũ giữ nguyên. | Chưa ghi | Chưa thực hiện |
| FR04-DT-16 | R3 | Gửi request với `role: "admin"` trong payload của User thường. | API bỏ qua/từ chối; role vẫn là `user`. | Chưa ghi | Chưa thực hiện |
| FR04-DT-17 | R3 | Sau khi cập nhật, đăng nhập lại hoặc GET hồ sơ để kiểm tra role. | User không được nâng thành Admin. | Chưa ghi | Chưa thực hiện |

## 6. Test xác thực và quyền truy cập

| Test ID | Lớp bao phủ | Điều kiện và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR04-DT-18 | A2 | Không gửi JWT khi gọi GET hồ sơ. | Bị từ chối xác thực. | Chưa ghi | Chưa thực hiện |
| FR04-DT-19 | A2 | Không gửi JWT khi gọi PUT cập nhật hồ sơ. | Bị từ chối; dữ liệu không đổi. | Chưa ghi | Chưa thực hiện |
| FR04-DT-20 | A3 | Gửi JWT sai hoặc hết hạn khi gọi GET/PUT hồ sơ. | Bị từ chối; dữ liệu không đổi. | Chưa ghi | Chưa thực hiện |
| FR04-DT-21 | A4 | Thử thay đổi ID hồ sơ trong request để truy cập User khác, nếu endpoint cho phép ID. | Bị từ chối hoặc không trả/cập nhật dữ liệu người khác. | Chưa ghi | Chưa thực hiện |

## 7. Tổng hợp thiết kế

| Nhóm | Số test case |
|---|---:|
| Đọc và cập nhật hồ sơ chính mình | 7 |
| Số điện thoại ngoài biên phân loại | 5 |
| Email và role bị giới hạn | 5 |
| Xác thực và quyền truy cập | 4 |
| **Tổng cộng** | **21** |

FR04 không lặp lại các test độ dài số điện thoại trong BVA. Tất cả test case bắt đầu ở trạng thái `Chưa thực hiện`.
