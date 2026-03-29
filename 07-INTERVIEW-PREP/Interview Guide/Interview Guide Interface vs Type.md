🎙️ Hướng dẫn trả lời phỏng vấn: Interface vs Type trong TypeScript

1. Định nghĩa ngắn gọn

Interface: Là một bản thiết kế (blueprint) để định nghĩa cấu trúc (shape) của một Object hoặc Class. Nó tập trung vào việc mô tả các "hợp đồng" dữ liệu.

Type Alias: Là một cách đặt tên (biệt danh) cho bất kỳ kiểu dữ liệu nào, từ Object, Primitive (string, number), Union, Tuple cho đến các logic tính toán type phức tạp.

2. 3 Điểm khác biệt mấu chốt (Kèm ví dụ)

A. Khả năng mở rộng (Extending)

Interface: Sử dụng từ khóa extends. TypeScript sẽ kiểm tra sự tương thích giữa các thuộc tính, nếu ghi đè sai kiểu dữ liệu sẽ báo lỗi ngay lập tức.

Type: Sử dụng dấu & (Intersection). Nếu có sự xung đột về kiểu dữ liệu, nó có thể tạo ra kiểu never mà không cảnh báo rõ ràng bằng Interface.

B. Gộp khai báo (Declaration Merging) - Điểm quan quan trọng nhất

Interface: Có tính năng Merging. Nếu khai báo 2 Interface cùng tên, chúng sẽ tự động gộp lại thành một. Điều này cực kỳ quan trọng khi viết Library hoặc mở rộng Global Types (như thêm thuộc tính vào window).

Type: Không được phép khai báo trùng tên.

C. Độ linh hoạt (Capability)

Interface: Chỉ dùng cho Object và Class.

Type: Có thể định nghĩa Union types (string | number), Tuples, hoặc Alias cho kiểu nguyên thủy. Interface không làm được điều này.

3. Lời khuyên thực chiến (The "Senior" Touch)

Đây là phần giúp bạn chứng tỏ mình có kinh nghiệm làm dự án thật:

"Trong các dự án Front-end thực tế, em thường áp dụng quy tắc sau:"

Ưu tiên dùng type cho:

React Component Props & State: Vì thường xuyên phải dùng Union types (ví dụ: status: 'loading' | 'success') và Intersection để kết hợp các thuộc tính HTML chuẩn.

Các kiểu dữ liệu trung gian, xử lý logic phức tạp (Mapped types, Conditional types).

Ưu tiên dùng interface cho:

Data Models từ API: Vì Interface có thông báo lỗi (error message) tường minh hơn khi làm việc với Object lớn.

Định nghĩa thư viện (Library): Để người dùng sau này có thể dùng Declaration Merging để mở rộng cấu trúc mà không cần sửa code gốc của mình.

4. Tóm tắt câu trả lời (Dành cho 1 phút nói)

"Dạ, cả Interface và Type đều dùng để định nghĩa hình dáng dữ liệu. Điểm khác biệt lớn nhất là Interface có thể tự động gộp các khai báo trùng tên (Merging) và chỉ dành cho Object/Class. Còn Type thì linh hoạt hơn, có thể đại diện cho Union, Tuple và các kiểu nguyên thủy.

Trong dự án React, em thường dùng Type cho Props và State vì sự linh hoạt, còn Interface em ưu tiên dùng cho các Model dữ liệu lớn hoặc khi viết các thư viện dùng chung để người khác dễ dàng mở rộng."
