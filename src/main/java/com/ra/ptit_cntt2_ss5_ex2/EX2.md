# [Bài tập 2]: Thiết kế REST API hệ thống mượn sách

## I. Mục tiêu
Thiết kế các endpoint RESTful cho chức năng quản lý Sách (Book) và Thẻ mượn (Loan) dựa trên các nguyên tắc chuẩn RESTful API.
* **Mỗi sách gồm:** `id`, `title`, `author`, `year`, `quantity`.
* **Mỗi thẻ mượn gồm:** `id`, `bookId`, `borrowerName`, `borrowDate`, `returnDate`.

---

## II. Bảng thiết kế REST API

Dưới đây là bảng thiết kế chi tiết hệ thống với ít nhất 8 dòng, bao gồm đầy đủ các thao tác CRUD và các đường dẫn tài nguyên phụ (sub-resource) hợp lý:

| Chức năng | Method | URL | Tham số / Request Body (nếu có) | Status thành công | Status lỗi (ví dụ) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Lấy danh sách sách** *(Có phân trang & lọc theo tác giả)* | **GET** | `/books` | **Query params:** `?page=1&limit=10&author=Nguyen+Nhat+Anh` | `200 OK` | `400 Bad Request` *(Sai định dạng tham số truyền vào)* |
| **2. Xem thông tin chi tiết một cuốn sách** | **GET** | `/books/:id` | *Không có* | `200 OK` | `404 Not Found` *(ID sách không tồn tại)* |
| **3. Thêm sách mới** | **POST** | `/books` | **Body JSON:** `{"title": "Mắt Biếc", "author": "Nguyễn Nhật Ánh", "year": 2019, "quantity": 15}` | `201 Created` | `400 Bad Request` *(Thiếu trường bắt buộc hoặc dữ liệu sai định dạng)* |
| **4. Cập nhật số lượng sách** *(Chỉ cập nhật 1 trường)* | **PATCH** | `/books/:id` | **Body JSON:** `{"quantity": 20}` | `200 OK` | `404 Not Found` *(ID sách không tồn tại)* |
| **5. Xóa sách khỏi hệ thống** | **DELETE** | `/books/:id` | *Không có* | `204 No Content` | `409 Conflict` *(Sách này đang có người mượn, không thể xóa)* |
| **6. Lấy danh sách thẻ mượn của một sách** *(Sub-resource)* | **GET** | `/books/:id/loans` | *Không có* | `200 OK` | `404 Not Found` *(ID sách không tồn tại)* |
| **7. Tạo thẻ mượn sách mới** | **POST** | `/loans` | **Body JSON:** `{"bookId": 1, "borrowerName": "Trần Văn B", "borrowDate": "2026-05-19"}` | `201 Created` | `400 Bad Request` *(Sách đã hết số lượng để cho mượn)* |
| **8. Trả sách** *(Cập nhật ngày trả thực tế)* | **PATCH** | `/loans/:id` | **Body JSON:** `{"returnDate": "2026-05-26"}` | `200 OK` | `404 Not Found` *(Mã số thẻ mượn không tồn tại)* |

---

## III. Ghi chú & Giải thích thiết kế

1. **Quy tắc đặt tên tài nguyên (URL):** Hệ thống sử dụng danh từ số nhiều `/books` và `/loans` xuyên suốt làm gốc, không lồng ghép động từ như `/getBooks` hay `/updateQuantity`.
2. **Cập nhật một phần (PATCH vs PUT):** Ở chức năng số 4 (Cập nhật số lượng sách) và chức năng số 8 (Trả sách), phương thức `PATCH` được ưu tiên sử dụng thay vì `PUT` do hệ thống chỉ thay đổi cục bộ 1-2 trường thông tin thay vì thay thế toàn bộ đối tượng tài nguyên ban đầu.
3. **Mối quan hệ cha - con (Sub-resource):** Endpoint `/books/:id/loans` thể hiện cấu trúc phân cấp tường minh của mô hình RESTful khi muốn tra cứu tất cả lịch sử giao dịch thẻ mượn gắn liền với một đầu sách cụ thể.