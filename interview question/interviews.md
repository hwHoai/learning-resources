# Bí kíp Phỏng vấn Full-stack & AI Integration (Fresher/Intern)

## 1. Web Fundamentals: GET vs POST
**Câu hỏi:** Phân biệt phương thức `GET` và `POST`? Khi nào bắt buộc dùng `POST`?
**Đáp án:**
Sự khác nhau cốt lõi nằm ở 3 điểm:
1. **Cách truyền & Dung lượng:** `GET` đính kèm dữ liệu vào URL nên bị giới hạn độ dài. `POST` đặt dữ liệu ngầm trong **Request Body**, cho phép gửi file lớn và an toàn hơn khi dùng HTTPS.
2. **Caching & Idempotent:** `GET` mang tính lũy đẳng (Idempotent - gọi nhiều lần không đổi trạng thái server) và có thể được trình duyệt cache (trả về 304). `POST` không được cache và không mang tính lũy đẳng.
3. **Mục đích:** `GET` dùng để lấy dữ liệu. Bắt buộc dùng `POST` khi gửi dữ liệu nhạy cảm (password) hoặc khi **tạo mới (create)** một entity.

## 2. React.js: State, Props & Data Flow
**Câu hỏi:** Khác biệt giữa `state` và `props`? Cách truyền dữ liệu từ Con lên Cha?
**Đáp án:**
1. `state` là dữ liệu cục bộ, có thể thay đổi (mutable) bên trong component. `props` là dữ liệu từ ngoài truyền vào, đối với component con thì nó là **chỉ đọc (read-only)**.
2. Để truyền dữ liệu từ Con lên Cha, sử dụng kỹ thuật **Lifting State Up** (hoặc Callback Function). Định nghĩa function ở component Cha, truyền qua `props` xuống component Con. Khi Con có sự kiện, gọi function đó và nạp dữ liệu vào tham số để Cha nhận được.

## 3. Backend Security: JSON Web Token (JWT)
**Câu hỏi:** Cấu tạo JWT? Tại sao không lưu mật khẩu vào payload của JWT?
**Đáp án:**
1. JWT gồm 3 phần: **Header** (loại token, thuật toán), **Payload** (dữ liệu user, thời hạn), và **Signature** (chữ ký xác thực toàn vẹn).
2. Tuyệt đối không lưu dữ liệu nhạy cảm vào Payload vì Header và Payload chỉ được mã hóa định dạng (**Base64Url Encode**) chứ không được mã hóa bảo mật (Encrypt). Bất kỳ ai cũng có thể decode và đọc được nội dung.

## 4. Database: SQL vs NoSQL
**Câu hỏi:** Tiêu chí cốt lõi để chọn SQL hay NoSQL cho dự án mới?
**Đáp án:**
Dựa vào 2 yếu tố cốt lõi:
1. **Cấu trúc & Tính toàn vẹn (ACID):** Nếu dữ liệu có quan hệ chặt chẽ, cần tính chính xác tuyệt đối (giao dịch, đơn hàng), ưu tiên **SQL** (PostgreSQL, MySQL).
2. **Linh hoạt & Mở rộng (Scaling):** Nếu dữ liệu không cố định (JSON-like), tốc độ đọc/ghi cao và cần dễ dàng mở rộng theo chiều ngang (Horizontal Scaling) như log, chat realtime, ưu tiên **NoSQL** (MongoDB).

## 5. JavaScript: Event Loop & Asynchronous
**Câu hỏi:** JS là đơn luồng (single-thread), tại sao gọi API tốn thời gian mà UI không bị đơ?
**Đáp án:**
Nhờ cơ chế **Event Loop** phối hợp với **Web APIs** (của trình duyệt) hoặc Hệ điều hành:
Khi Call Stack gặp lệnh gọi API (fetch), nó giao việc đó cho Web APIs xử lý ngầm và tiếp tục chạy code khác (không chặn luồng). Khi API có kết quả, hàm xử lý (callback) được đẩy vào **Callback Queue**. **Event Loop** sẽ liên tục kiểm tra, khi Call Stack rảnh mới đưa callback từ Queue lên để thực thi.

## 6. Git & Teamwork: Xử lý code dở dang
**Câu hỏi:** Đang code dở nhánh `feature-A`, sếp bắt sang `main` fix bug gấp. Xử lý sao để không làm rác commit mà không mất code?
**Đáp án:**
Sử dụng lệnh **`git stash`**.
1. Tại nhánh `feature-A`, chạy `git stash` để lưu tạm các thay đổi vào bộ nhớ tạm, trả working directory về trạng thái sạch (clean).
2. Checkout sang `main` để xử lý bug và commit.
3. Quay lại nhánh `feature-A`, chạy `git stash pop` để bung phần code dở dang ra và làm tiếp.

## 7. Web Security: Lỗi CORS
**Câu hỏi:** Lỗi "Blocked by CORS policy" bản chất là gì? Cấu hình sửa ở đâu?
**Đáp án:**
1. **Bản chất:** CORS (Cross-Origin Resource Sharing) là cơ chế bảo vệ của **Trình duyệt**, tự động chặn các request gọi API khác nguồn gốc (khác domain/port) để bảo vệ người dùng khỏi trang web độc hại.
2. **Cách sửa:** Dù trình duyệt chặn, nhưng phải **cấu hình cấp phép ở phía Backend**. Backend cần thiết lập các Header (`Access-Control-Allow-Origin`) và định nghĩa rõ Method nào được phép. Đưa các URL Frontend hợp lệ (Whitelist) vào file `.env` thay vì dùng dấu `*`.

## 8. Performance Optimization: Handle Large Data
**Câu hỏi:** Tải 10,000 sản phẩm kèm ảnh nặng, làm sao để web không sập/đơ?
**Đáp án:**
Phải xử lý đồng bộ cả 2 phía:
1. **Backend:** Dùng **Pagination** (phân trang) hoặc Cursor, đánh **Index** DB, và đặc biệt chỉ trả về link ảnh **Thumbnail** (thu nhỏ) thay vì ảnh gốc cho danh sách.
2. **Frontend:** Áp dụng **Lazy Loading** (cuộn tới đâu tải ảnh tới đó), kết hợp **Prefetching/Caching** (tải ngầm trước trang kế tiếp) để tăng trải nghiệm mượt mà.

## 9. Deployment & DevOps: Docker
**Câu hỏi:** CI/CD là gì? Lợi ích cốt lõi của Docker so với việc copy code lên server?
**Đáp án:**
1. **Luồng CI/CD cơ bản:** Đẩy code lên GitHub -> GitHub Actions tự động build, tạo **Docker Image**, push lên Docker Hub -> Server pull Image về chạy thành **Container**.
2. **Lợi ích Docker:** Mang lại tính **Nhất quán (Consistency) và Cô lập (Isolation)**. Nó đóng gói toàn bộ code và môi trường chạy vào một chỗ, loại bỏ hoàn toàn lỗi "chạy được trên máy dev nhưng tịt trên server" hay xung đột phiên bản phần mềm.

## 10. AI Integration: Troubleshooting Pipeline
**Câu hỏi:** Model AI test độc lập (Jupyter) rất chuẩn, nhưng tích hợp vào App thực tế (UI truyền dữ liệu xuống) lại sai bét. Kiểm tra yếu tố nào đầu tiên?
**Đáp án:**
Kiểm tra tính nhất quán của dữ liệu đầu vào (khâu Feature Extraction) để loại trừ hiện tượng **Training-Serving Skew**.
Giải pháp: Đặt log in ra dữ liệu ngay trước dòng `model.predict()` ở cả Jupyter và source code App. So sánh đối chiếu để đảm bảo logic trích xuất từ giao diện sinh ra đúng định dạng (shape), kiểu dữ liệu (data type) và dải giá trị (scale) y hệt như lúc huấn luyện.

# Bí kíp Phỏng vấn Hiệp 2: Core, Framework & Architecture

## 11. TypeScript: Interface vs Type
**Bản chất:** `interface` dùng cho Object/Class, hỗ trợ gộp khai báo (Merging). `type` đa năng hơn, dùng cho cả Union types, Primitives.
**Thực chiến:** Dùng `interface` cho Props, DTO, State để hướng đối tượng. Dùng `type` cho các kiểu dữ liệu kết hợp phức tạp.

## 12. TypeScript: Generics <T>
**Mục đích:** Tái sử dụng code (Reusability) mà vẫn giữ được tính chặt chẽ về kiểu (Type-safety). 
**Ứng dụng:** Dùng cho các hàm gọi API dùng chung, các Component UI (Table, Select) để IDE có thể gợi ý đúng thuộc tính của dữ liệu truyền vào thay vì dùng `any`.

## 13. OOP: Tính Đa hình (Polymorphism)
**Bản chất:** Một giao diện (Interface), nhiều cách thực thi. Giúp code chính không cần quan tâm chi tiết class con nào đang chạy.
**Ứng dụng:** Hệ thống thanh toán (Momo, ZaloPay cùng implement IPayment). Giúp hệ thống dễ mở rộng (Open/Closed Principle) mà không làm hỏng logic cũ.

## 14. React: useEffect Lifecycle
**Quy tắc:** `render` (vẽ giao diện) khác với `mount` (gắn vào DOM).
- `[]`: Chạy 1 lần sau lần render đầu (Init dữ liệu).
- `[dep]`: Chạy lại mỗi khi `dep` thay đổi (Filter, Search).
- No array: Chạy sau mọi lần render (Dễ gây loop, hạn chế dùng).

## 15. SQL: Joins
- **INNER JOIN:** Chỉ lấy phần giao (User phải có đơn hàng mới hiện).
- **LEFT JOIN:** Lấy toàn bộ bảng trái (Hiện tất cả User, ai chưa mua thì đơn hàng là NULL).
- **SELF JOIN:** Bảng tự join chính nó (Dùng cho cấu trúc sếp - nhân viên).

## 16. DSA: Graph & BFS
- **Graph (Đồ thị):** Cấu trúc tốt nhất cho mạng xã hội (Node = User, Edge = Mối quan hệ).
- **BFS (Duyệt chiều rộng):** Thuật toán tìm đường ngắn nhất (như tìm bạn chung) nhờ cơ chế quét theo từng lớp layer-by-layer.

## 17. Node.js: CPU-bound vs Worker Threads
**Vấn đề:** Tác vụ nặng (nén video) làm block Main Thread (đơn luồng). `async/await` không giải quyết được vì nó dành cho I/O.
**Giải pháp:** Dùng `worker_threads` để đẩy tác vụ sang luồng riêng, tận dụng đa nhân CPU mà không làm treo Server.

## 18. React: State Batching & Updater
**Cơ chế:** React gom nhiều lệnh `setCount` lại để render 1 lần (Batching). State trong 1 lần render là "ảnh chụp tĩnh" (Snapshot).
**Giải pháp:** Dùng updater function `setCount(prev => prev + 1)` để lấy giá trị mới nhất trong hàng đợi thay vì giá trị snapshot cũ.

## 19. Next.js: SSR vs CSR
**SSR (Server-Side Rendering):** Gọi API và render HTML ngay tại Server.
**Lợi ích:** Tối ưu SEO (Google thấy nội dung ngay) và tăng tốc độ hiển thị đầu tiên (FCP), cực kỳ quan trọng cho trang bán hàng/tin tức.

## 20. Database: Indexing (B-Tree)
**Cơ chế:** Dùng cây B-Tree để tìm kiếm nhanh (O(log n)).
**Lưu ý:** Không đánh Index bừa bãi vì làm chậm lệnh Ghi (INSERT/UPDATE/DELETE) và tốn bộ nhớ RAM/Disk.