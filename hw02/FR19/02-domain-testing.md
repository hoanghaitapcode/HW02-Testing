# FR-19 — Domain Testing quản lý người dùng Admin

## 1. Phạm vi

FR-19 kiểm tra hai nhóm chức năng:

1. Admin xem danh sách tất cả người dùng nhưng không được lộ mật khẩu.
2. Admin xóa được người dùng khác nhưng không được xóa chính tài khoản đang đăng nhập.

Không thêm cột `Bằng chứng`; kết quả ảnh lỗi được quản lý riêng trong bug report nếu phát hiện lỗi.

## 2. Biến miền và lớp tương đương

### 2.1. Actor và xác thực

| Mã lớp | Điều kiện | Kết quả kỳ vọng |
|---|---|---|
| A1 | Admin có JWT hợp lệ và `role=admin` | Được thực hiện thao tác Admin hợp lệ |
| A2 | User thường có JWT hợp lệ | Bị từ chối API Admin |
| A3 | Không có JWT | Bị từ chối xác thực |
| A4 | JWT sai hoặc hết hạn | Bị từ chối xác thực |
| A5 | JWT có nhưng role không phải `admin` | Bị từ chối phân quyền |

### 2.2. Đối tượng người dùng

| Mã lớp | Đối tượng | Kết quả kỳ vọng |
|---|---|---|
| U1 | Danh sách có nhiều người dùng | Admin xem được các tài khoản trong phạm vi hệ thống |
| U2 | Tài khoản khác với tài khoản Admin đang đăng nhập | Có thể xóa nếu tài khoản tồn tại |
| U3 | Chính tài khoản Admin đang đăng nhập | Không được xóa |
| U4 | ID người dùng không tồn tại | Trả lỗi phù hợp; không xóa dữ liệu khác |

### 2.3. Dữ liệu nhạy cảm

| Mã lớp | Dữ liệu trả về | Kết quả kỳ vọng |
|---|---|---|
| D1 | Bản ghi có trường định danh, email, tên, role | Hiển thị các trường được phép |
| D2 | Bản ghi có `password` hoặc biến thể chứa mật khẩu | Không được trả về cho client |

## 3. Test danh sách người dùng

| Test ID | Lớp bao phủ | Điều kiện và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR19-DT-01 | A1, U1 | Đăng nhập Admin và mở danh sách người dùng. | Hiển thị danh sách tất cả người dùng trong hệ thống. | Hiển thị danh sách người dùng | Pass|
| FR19-DT-02 | A1, D1 | Quan sát một bản ghi người dùng trong danh sách Admin. | Hiển thị thông tin được phép như email, tên và role. | Hiển thị email, role | Pass |
| FR19-DT-03 | A1, D2 | Kiểm tra response API hoặc dữ liệu hiển thị của danh sách. | Không có trường `password`, mật khẩu plaintext hoặc biến thể tương đương. | Không có password trong response | Pass |
| FR19-DT-04 | A2 | Đăng nhập User thường và gọi API/xem trang danh sách người dùng Admin. | Bị từ chối; không lộ danh sách người dùng. | UI không hỗ trợ  | Block |
| FR19-DT-05 | A3 | Không gửi JWT rồi gọi API danh sách người dùng. | Bị từ chối xác thực. | HTTP trả 401 | Pass |
| FR19-DT-06 | A4 | Gọi API bằng JWT sai hoặc hết hạn. | Bị từ chối xác thực; không trả danh sách. |HTTP code trả 403 | Pass |
| FR19-DT-07 | A5 | Gọi API bằng JWT hợp lệ nhưng role là User. | Bị từ chối quyền Admin; không trả danh sách. | HTTP trả 401 | Pass |

## 4. Test xóa người dùng

Tiền điều kiện: có tài khoản Admin đang đăng nhập và có ít nhất một tài khoản User khác để thử xóa. Nếu không có User thứ hai, test xóa người khác phải ghi `Blocked`, không tự dùng tài khoản Admin làm đối tượng khác.

| Test ID | Lớp bao phủ | Điều kiện và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR19-DT-08 | A1, U2 | Admin xóa một User khác đang tồn tại. | Xóa thành công; User không còn trong danh sách. | User không còn tron danh sách | Pass |
| FR19-DT-09 | A1, U3 | Admin gửi yêu cầu xóa chính user ID của tài khoản đang đăng nhập. | Bị từ chối; tài khoản Admin vẫn hoạt động và còn trong danh sách. | User vẫn bị xóa khỏi danh sách |  |
| FR19-DT-10 | A1, U4 | Admin xóa một user ID không tồn tại. | Trả lỗi phù hợp; không xóa nhầm tài khoản khác. | Vẫn xóa được user | Fail |
| FR19-DT-11 | A2, U2 | User thường gọi API xóa một User khác. | Bị từ chối do không có quyền Admin; dữ liệu không đổi. | Vẫn xóa được user với status code 200 | Fail |
| FR19-DT-12 | A3, U2 | Không có JWT gọi API xóa User khác. | Bị từ chối xác thực; dữ liệu không đổi. | Bị từ chối | Pass |
| FR19-DT-13 | A4, U2 | Dùng JWT sai hoặc hết hạn để xóa User khác. | Bị từ chối xác thực; dữ liệu không đổi. | Không xóa được | Pass |
| FR19-DT-14 | A5, U2 | Dùng JWT User hợp lệ không có role admin để xóa User khác. | Bị từ chối phân quyền; dữ liệu không đổi. | Vẫn xóa được | Fail |

## 5. Tính toàn vẹn sau thao tác

| Test ID | Lớp bao phủ | Điều kiện và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR19-DT-15 | U3 | Sau khi thử xóa chính mình, Admin tải lại danh sách và thực hiện thao tác Admin khác. | Admin vẫn tồn tại, token còn hoạt động và thao tác Admin tiếp theo được xử lý bình thường. | Xóa được admin nhưng refresh FE vẫn không mất token | Fail |
| FR19-DT-16 | U2 | Sau khi xóa User khác, tải lại danh sách và gọi lại API lấy danh sách. | User đã xóa không còn xuất hiện; các User khác không bị ảnh hưởng. | Xóa thành công và hiển thị đúng | Pass |

## 6. Các điểm chưa được đặc tả rõ

Các điểm sau không tự chuyển thành Fail nếu chưa có yêu cầu bổ sung:

- Admin có được xóa một Admin khác hay không; đặc tả chỉ cấm xóa chính tài khoản đang đăng nhập.
- Hệ thống có yêu cầu xác nhận trước khi xóa hay không.
- Mã HTTP và nội dung chính xác của từng thông báo lỗi.
- Phân trang, tìm kiếm và giới hạn số User trong danh sách.

## 7. BVA

FR-19 không có giới hạn số được nêu rõ trong đặc tả như độ dài, số lượng trang hoặc kích thước danh sách. Vì vậy chưa tạo BVA số giả định; các biên chính của feature được kiểm tra bằng Domain Testing: Admin/User, chính mình/người khác, tồn tại/không tồn tại và có/không có xác thực.

## 8. Tổng hợp

| Nhóm | Số test case |
|---|---:|
| Xem danh sách và bảo mật dữ liệu | 7 |
| Xóa người dùng | 7 |
| Tính toàn vẹn sau thao tác | 2 |
| **Tổng cộng** | **16** |

Tất cả test case bắt đầu ở trạng thái `Chưa thực hiện`.
