---
title: "Blog 2: Single-Table Design Trong Amazon DynamoDB"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Tư Duy Thiết Kế Single-Table Design Trên Amazon DynamoDB: Tối Ưu Hiệu Năng Và Chi Phí Cho Hệ Thống Serverless

---

Trong thế giới phát triển ứng dụng đám mây hiện đại, **Amazon DynamoDB** được biết đến như một dịch vụ cơ sở dữ liệu NoSQL quản lý hoàn toàn (fully managed) có khả năng duy trì độ trễ phản hồi dưới 10 milliseconds ở bất kỳ quy mô dữ liệu nào — dù hệ thống của bạn chứa vài gigabyte hay hàng trăm terabyte.

Tuy nhiên, rất nhiều lập trình viên khi chuyển từ cơ sở dữ liệu quan hệ (RDBMS) sang DynamoDB đã mắc phải sai lầm lớn: Mang tư duy thiết kế đa bảng (Multi-Table) và chuẩn hóa dữ liệu (Normalization) áp dụng vào NoSQL. Bài viết này sẽ phân tích chi tiết tư duy thiết kế **Single-Table Design** (Thiết kế một bảng duy nhất) — kỹ thuật tối thượng giúp khai thác tối đa sức mạnh của DynamoDB.

---

### 1. Bối Cảnh: Sự Khác Biệt Bản Chất Giữa RDBMS Và DynamoDB

Để hiểu lý do tại sao Single-Table Design ra đời, chúng ta cần nhìn lại sự khác biệt về cách vận hành giữa hai mô hình cơ sở dữ liệu:

- **Mô hình RDBMS (MySQL, PostgreSQL)**: Tối ưu hóa cho chi phí lưu trữ đĩa (Storage Cost). Dữ liệu được chia nhỏ và chuẩn hóa vào nhiều bảng riêng biệt (`Users`, `Orders`, `OrderItems`). Khi cần lấy toàn bộ thông tin đơn hàng của người dùng, hệ thống sử dụng các phép nối `JOIN`. Khi dữ liệu phình to lên hàng triệu bản ghi, việc thực thi `JOIN` đòi hỏi CPU quét qua nhiều chỉ mục trên các ổ đĩa khác nhau, gây nghẽn hiệu năng và tăng độ trễ rõ rệt.
- **Mô hình DynamoDB (NoSQL)**: Tối ưu hóa cho hiệu năng truy vấn (Compute Cost & Latency). DynamoDB không hỗ trợ phép nối `JOIN`. Mọi dữ liệu cần thiết cho một màn hình ứng dụng phải được truy xuất chỉ trong một đợt đọc duy nhất (Single Request).

Do đó, nguyên tắc vàng trong DynamoDB là: **Phải xác định toàn bộ các mẫu truy vấn (Access Patterns) trước khi thiết kế cấu hình bảng.**

---

### 2. Nguyên Lý Cốt Lõi Của Single-Table Design

Kỹ thuật Single-Table Design là việc đưa tất cả các đối tượng thực thể có quan hệ với nhau (ví dụ: Thông tin người dùng, Đơn hàng, Chi tiết sản phẩm, Hóa đơn) vào lưu trữ chung trong một bảng DynamoDB duy nhất.

#### Cách Thức Hoạt Động Thông Qua Khóa Tổng Quát (Generic Keys)

Thay vì đặt tên cột khóa chính cố định như `UserId` hay `OrderId`, một bảng Single-Table Design sẽ đặt tên khóa chính ở dạng tổng quát:

- **Partition Key (PK)**: Định vị phân vùng lưu trữ vật lý của dữ liệu.
- **Sort Key (SK)**: Sắp xếp các bản ghi trong cùng một phân vùng và phân loại loại thực thể.

Bằng cách sử dụng các chuỗi tiền tố (Prefix / Namespace) kết hợp với ký tự phân cách (như `#`), chúng ta tiến hành **Overloading Keys** (chồng hồ sơ khóa). Ví dụ:

- **Dữ liệu Người dùng (User Profile)**: Đặt `PK = USER#1001` và `SK = METADATA#1001`.
- **Dữ liệu Đơn hàng của người dùng đó (User Order)**: Đặt `PK = USER#1001` và `SK = ORDER#2026#001`.
- **Chi tiết mặt hàng trong đơn hàng (Order Item)**: Đặt `PK = ORDER#2026#001` và `SK = ITEM#SKU-992`.

#### Khái Niệm Item Collections (Tập Hợp Bản Ghi Đa Dạng)

Tất cả các bản ghi có chung thuộc tính `PK = USER#1001` sẽ nằm trên cùng một phân vùng ổ đĩa và tạo thành một **Item Collection**.

Khi ứng dụng cần hiển thị trang thông tin cá nhân kèm theo lịch sử mua hàng của người dùng 1001, thay vì phải gửi 2 request riêng biệt đến 2 bảng khác nhau, ứng dụng chỉ cần gửi một lệnh `Query` duy nhất với điều kiện `PK = USER#1001`. DynamoDB sẽ trả về cả thông tin cá nhân lẫn danh sách tất cả các đơn hàng chỉ trong vài milliseconds.

---

### 3. Quản Lý Truy Vấn Hai Chiều Bằng Global Secondary Index (GSI)

Một thách thức lớn của Single-Table Design là: Làm sao để truy vấn ngược lại? (Ví dụ: Bạn có `OrderId = ORDER#2026#001` và muốn tìm xem đơn hàng này thuộc về `UserId` nào).

Trong Single-Table Design, giải pháp là sử dụng **Global Secondary Index (GSI)**:

- Chúng ta tạo một chỉ mục phụ **GSI-1** với thuộc tính đảo ngược: `GSI1-PK = SK` và `GSI1-SK = PK`.
- Khi cần tìm thông tin chủ sở hữu đơn hàng `ORDER#2026#001`, chúng ta thực hiện `Query` trên GSI-1 với điều kiện `GSI1-PK = ORDER#2026#001`.

Chỉ mục GSI sẽ tự động nhân bản dữ liệu theo thời gian thực (asynchronous replication) để phục vụ các luồng truy vấn phụ mà không làm ảnh hưởng tới hiệu năng của bảng chính.

---

### 4. Lợi Ích Vượt Trội Của Single-Table Design

- **Tiết Kiệm Chi Phí Vận Hành (RCU / WCU)**: Trong DynamoDB, chi phí được tính dựa trên đơn vị năng lực đọc/ghi (Read/Write Capacity Units). Việc gom dữ liệu vào một bảng giúp bạn dễ dàng thiết lập Auto-scaling hoặc Provisioned Capacity tập trung, tránh lãng phí năng lực xử lý ở các bảng ít dùng.
- **Tốc Độ Truy Xuất Tối Ưu**: Giảm thiểu tối đa số lượng kết nối mạng (Network Round-trips). Một lệnh gọi duy nhất lấy về toàn bộ cấu trúc dữ liệu phức tạp.
- **Tự Động Mở Rộng Linh Hoạt**: Bảng DynamoDB đơn duy nhất có thể tăng trưởng từ vài nghìn request/giây lên hàng triệu request/giây mà không cần thay đổi kiến trúc hạ tầng hay can thiệp thủ công.

---

### 5. Khi Nào Nên Và Không Nên Áp Dụng Single-Table Design?

#### Nên Áp Dụng Khi:

- Ứng dụng Serverless quy mô vừa và lớn có các đường truy vấn (Access Patterns) đã được xác định rõ ràng từ khâu thiết kế.
- Hệ thống có lượng truy cập cực cao, nhạy cảm với độ trễ phản hồi (như ứng dụng thương mại điện tử, thanh toán tài chính, game online).
- Cần tối ưu hóa ngân sách hạ tầng AWS.

#### Không Nên Áp Dụng Khi:

- Hệ thống yêu cầu các truy vấn linh hoạt ngẫu nhiên (Ad-hoc queries) hoặc phục vụ phân tích dữ liệu kinh doanh (BI / Analytics) không cố định.
- Đội ngũ phát triển mới làm quen với NoSQL (nên bắt đầu bằng mô hình Multi-Table đơn giản hơn để tránh thiết kế sai khóa PK/SK).

---

### Kết Luận & Tài Liệu Tham Khảo

Chuyển đổi từ tư duy RDBMS đa bảng sang Single-Table Design trên Amazon DynamoDB đòi hỏi một góc nhìn hoàn toàn mới về cấu trúc dữ liệu. Bằng việc gom nhóm các thực thể theo phân vùng (PK) và phân loại theo khóa sắp xếp (SK), doanh nghiệp sở hữu một nền tảng cơ sở dữ liệu vững chắc, sẵn sàng chịu tải khổng lồ với chi phí tối ưu nhất.

**Tài liệu tham khảo chính thức từ AWS:**

- Amazon DynamoDB Developer Guide: [Best Practices for Designing DynamoDB Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
