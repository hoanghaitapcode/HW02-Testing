# FR-10 — Domain Testing trạng thái đơn hàng

## 1. Cách áp dụng Domain Testing

Các biến miền của chức năng:

- Trạng thái đầu: `pending`, `confirmed`, `shipping`, `delivered`, `canceled`.
- Trạng thái đích: năm trạng thái trên.
- Actor: User sở hữu đơn, User không sở hữu đơn, Admin, người chưa xác thực.
- Quyền xác thực: JWT hợp lệ, thiếu JWT, JWT không hợp lệ, JWT không có role admin.
- Quan hệ sở hữu: đơn của chính User hoặc đơn của User khác.

Mỗi test case kiểm tra một chuyển đổi hoặc một điều kiện quyền chính. Sau thao tác thất bại, trạng thái đơn phải giữ nguyên.

## 2. Phân vùng trạng thái

| Mã lớp | Miền | Giá trị |
|---|---|---|
| S1 | Trạng thái đầu | `pending` |
| S2 | Trạng thái đầu | `confirmed` |
| S3 | Trạng thái đầu | `shipping` |
| S4 | Trạng thái đầu kết thúc | `delivered` |
| S5 | Trạng thái đầu kết thúc | `canceled` |
| D1 | Trạng thái đích | `pending` |
| D2 | Trạng thái đích | `confirmed` |
| D3 | Trạng thái đích | `shipping` |
| D4 | Trạng thái đích | `delivered` |
| D5 | Trạng thái đích | `canceled` |

## 3. Phân vùng actor và quyền

| Mã lớp | Actor / điều kiện | Kết quả quyền kỳ vọng |
|---|---|---|
| A1 | Admin có JWT hợp lệ và role `admin` | Được thực hiện thao tác quản trị hợp lệ |
| A2 | User có JWT hợp lệ, là chủ đơn | Chỉ được hủy đơn `pending` hoặc `confirmed` |
| A3 | User có JWT hợp lệ, không phải chủ đơn | Từ chối thao tác trên đơn của người khác |
| A4 | Không có JWT | Từ chối yêu cầu cần xác thực |
| A5 | JWT không hợp lệ hoặc hết hạn | Từ chối yêu cầu |
| A6 | JWT hợp lệ nhưng không có role `admin` khi gọi thao tác admin | Từ chối quyền admin |

## 4. Ma trận chuyển đổi

| Trạng thái đầu | `pending` | `confirmed` | `shipping` | `delivered` | `canceled` |
|---|---|---|---|---|---|
| `pending` | Không đặc tả | Hợp lệ: Admin | Hợp lệ: Admin | Không hợp lệ | Hợp lệ: User/Admin |
| `confirmed` | Không hợp lệ | Không đặc tả | Hợp lệ: Admin | Không hợp lệ | Hợp lệ: User/Admin |
| `shipping` | Không hợp lệ | Không hợp lệ | Không đặc tả | Hợp lệ: Admin | User: không hợp lệ; Admin: chưa đặc tả rõ |
| `delivered` | Không hợp lệ | Không hợp lệ | Không hợp lệ | Không đặc tả | Không hợp lệ |
| `canceled` | Không hợp lệ | Không hợp lệ | Không hợp lệ | Không hợp lệ | Không đặc tả |

Các ô cùng trạng thái là `Ambiguous` vì đặc tả chưa nói rõ yêu cầu gửi lại cùng trạng thái. `shipping → canceled` đối với Admin cũng là `Ambiguous`; không tự xem đó là bug nếu chưa có quyết định yêu cầu.

## 5. Chuyển đổi hợp lệ

| Test ID | Lớp bao phủ | Điều kiện và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR10-DT-01 | S1→D2, A1 | Admin chuyển `pending` sang `confirmed`. | Thành công; trạng thái mới là `confirmed`. | Trạng thái đổi sang confirmed | Pass |
| FR10-DT-02 | S1→D5, A2 | Chủ đơn hủy đơn `pending`. | Thành công; trạng thái mới là `canceled`. | Không thấy lịch sử đặt hàng để hủy | Blocked |
| FR10-DT-03 | S1→D5, A1 | Admin hủy đơn `pending`. | Thành công; trạng thái mới là `canceled`. | Hủy thành công | Pass |
| FR10-DT-04 | S2→D3, A1 | Admin chuyển `confirmed` sang `shipping`. | Thành công; trạng thái mới là `shipping`. | Chuyển trạng thái thành shipping | Pass |
| FR10-DT-05 | S2→D5, A2 | Chủ đơn hủy đơn `confirmed`. | Thành công; trạng thái mới là `canceled`. | Không thấy lịch sử đặt hàng để hủy | Blocked |
| FR10-DT-06 | S2→D5, A1 | Admin hủy đơn `confirmed`. | Thành công; trạng thái mới là `canceled`. | Hủy thành công | Pass |
| FR10-DT-07 | S3→D4, A1 | Admin chuyển `shipping` sang `delivered`. | Thành công; trạng thái mới là `delivered`. | Chuyển thành công | Pass |

## 6. Chuyển đổi không hợp lệ

| Test ID | Lớp bao phủ | Điều kiện và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR10-DT-08 | S1→D3 | Chuyển `pending` sang `shipping`. | Trả lỗi; trạng thái vẫn `pending`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-09 | S1→D4 | Chuyển `pending` sang `delivered`. | Trả lỗi; trạng thái vẫn `pending`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-10 | S2→D1 | Chuyển `confirmed` về `pending`. | Trả lỗi; trạng thái vẫn `confirmed`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-11 | S2→D4 | Chuyển `confirmed` sang `delivered`. | Trả lỗi; trạng thái vẫn `confirmed`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-12 | S3→D1 | Chuyển `shipping` về `pending`. | Trả lỗi; trạng thái vẫn `shipping`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-13 | S3→D2 | Chuyển `shipping` về `confirmed`. | Trả lỗi; trạng thái vẫn `shipping`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-14 | S3→D5, A2 | User cố hủy đơn đang `shipping`. | Từ chối; trạng thái vẫn `shipping`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-15 | S4→D1 | Chuyển `delivered` sang `pending`. | Trả lỗi; trạng thái vẫn `delivered`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-16 | S4→D2 | Chuyển `delivered` sang `confirmed`. | Trả lỗi; trạng thái vẫn `delivered`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-17 | S4→D3 | Chuyển `delivered` sang `shipping`. | Trả lỗi; trạng thái vẫn `delivered`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-18 | S4→D5 | Chuyển `delivered` sang `canceled`. | Trả lỗi; trạng thái vẫn `delivered`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-19 | S5→D1 | Chuyển `canceled` sang `pending`. | Trả lỗi; trạng thái vẫn `canceled`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-20 | S5→D2 | Chuyển `canceled` sang `confirmed`. | Trả lỗi; trạng thái vẫn `canceled`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-21 | S5→D3 | Chuyển `canceled` sang `shipping`. | Trả lỗi; trạng thái vẫn `canceled`. | UI chỉ hỗ trợ chuyển step by step | Block |
| FR10-DT-22 | S5→D4 | Chuyển `canceled` sang `delivered`. | Trả lỗi; trạng thái vẫn `canceled`. | UI chỉ hỗ trợ chuyển step by step | Block |

## 7. Quyền, xác thực và sở hữu

| Test ID | Lớp bao phủ | Điều kiện và thao tác | Kết quả mong đợi | Kết quả thực tế | Trạng thái |
|---|---|---|---|---|---|
| FR10-DT-23 | A3 | User A dùng tài khoản của mình hủy đơn `pending` thuộc User B. | Từ chối; trạng thái đơn không đổi. | Chưa ghi | Chưa thực hiện |
| FR10-DT-24 | A4 | Không gửi JWT khi chuyển đơn `pending` sang `confirmed`. | Từ chối xác thực; trạng thái không đổi. | Chưa ghi | Chưa thực hiện |
| FR10-DT-25 | A5 | Gửi JWT sai hoặc hết hạn để chuyển `pending` sang `confirmed`. | Từ chối xác thực; trạng thái không đổi. | Chưa ghi | Chưa thực hiện |
| FR10-DT-26 | A6 | User có JWT hợp lệ nhưng không có role admin gọi thao tác Admin `pending`→`confirmed`. | Từ chối quyền; trạng thái không đổi. | Chưa ghi | Chưa thực hiện |
| FR10-DT-27 | A2 | Chủ đơn cố chuyển `pending`→`confirmed` bằng quyền User. | Từ chối vì chỉ Admin được xác nhận; trạng thái không đổi. | Chưa ghi | Chưa thực hiện |
| FR10-DT-28 | A2 | Chủ đơn cố chuyển `confirmed`→`shipping` bằng quyền User. | Từ chối vì chỉ Admin được giao hàng; trạng thái không đổi. | Chưa ghi | Chưa thực hiện |

## 8. Các trường hợp chưa đặc tả rõ

Các test sau được thiết kế để hỏi/ghi nhận hành vi, nhưng không tự kết luận Pass/Fail khi chưa có quyết định yêu cầu:

| Test ID | Trường hợp | Kết quả cần ghi |
|---|---|---|
| FR10-DT-29 | Admin chuyển `shipping`→`canceled`. | `Ambiguous` nếu đặc tả chưa xác nhận quyền này. |
| FR10-DT-30 | Gửi yêu cầu `pending`→`pending`. | `Ambiguous` vì chưa đặc tả thao tác cùng trạng thái. |
| FR10-DT-31 | Gửi yêu cầu `confirmed`→`confirmed`. | `Ambiguous` vì chưa đặc tả thao tác cùng trạng thái. |
| FR10-DT-32 | Gửi yêu cầu `shipping`→`shipping`. | `Ambiguous` vì chưa đặc tả thao tác cùng trạng thái. |
| FR10-DT-33 | Gửi yêu cầu `delivered`→`delivered`. | `Ambiguous` vì chưa đặc tả thao tác cùng trạng thái. |
| FR10-DT-34 | Gửi yêu cầu `canceled`→`canceled`. | `Ambiguous` vì chưa đặc tả thao tác cùng trạng thái. |

## 9. Tổng hợp thiết kế

| Nhóm | Số test case |
|---|---:|
| Chuyển đổi hợp lệ | 7 |
| Chuyển đổi không hợp lệ | 15 |
| Quyền, xác thực và sở hữu | 6 |
| Trường hợp chưa đặc tả rõ | 6 |
| **Tổng cộng** | **34** |

Các test case bắt đầu ở trạng thái `Chưa thực hiện`. Không có BVA số riêng cho FR-10 vì các trạng thái là miền phân loại; các biên phân loại đã được bao phủ bằng Domain Testing.
