---
title: "Các bài blogs đã đăng"
date: 2026-08-04
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

# Danh Sách Các Bài Viết Kỹ Thuật Đã Đăng

Mục này tổng hợp các bài viết chia sẻ kiến thức chuyên sâu về công nghệ Điện toán đám mây AWS được thực hiện trong kỳ thực tập và chia sẻ trên cộng đồng [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj):

---

### 1. [Blog 1 - Tối Ưu Hiệu Năng & Xử Lý Cold Start Trong AWS Lambda](3.1-Blog1/)
Bài viết phân tích bản chất môi trường thực thi Serverless Compute, chu kỳ sống 3 giai đoạn (Init Phase, Invoke Phase, Shutdown Phase) của AWS Lambda và nguyên nhân gây ra độ trễ khởi tạo (Cold Start). Bài viết trình bày chi tiết kỹ thuật tái sử dụng kết nối (Execution Context Reuse) bằng cách khởi tạo đối tượng SDK/CSDL ở phạm vi toàn cục (Global Scope) để giảm thời gian phản hồi từ hàng trăm milliseconds xuống dưới 10 milliseconds.

---

### 2. [Blog 2 - Tư Duy Thiết Kế Single-Table Design Trên Amazon DynamoDB](3.2-Blog2/)
Bài viết chia sẻ góc nhìn chuyển đổi từ tư duy CSDL quan hệ (RDBMS) sang NoSQL trên Amazon DynamoDB. Nội dung đi sâu phân tích kỹ thuật thiết kế một bảng duy nhất (Single-Table Design) thông qua việc chồng hồ sơ khóa (Key Overloading với PK/SK), tạo tập hợp bản ghi (Item Collections) và sử dụng Global Secondary Index (GSI) để phục vụ truy vấn 2 chiều với chi phí RCU/WCU tối ưu nhất.

---

### 3. [Blog 3 - Xây Dựng Lớp Bảo Vệ & Điều Phối Lưu Lượng Với Amazon API Gateway](3.3-Blog3/)
Bài viết phân tích vai trò của Amazon API Gateway như một "người gác cổng" (Reverse Proxy) tập trung bảo vệ các hệ thống Serverless backend khỏi rủi ro bị tấn công DDoS và quá tải hạ tầng. Bài viết phân tích 4 trụ cột công nghệ cốt lõi: Giới hạn tốc độ theo thuật toán Token Bucket (Rate Limiting & Throttling), Xác thực tập trung với Lambda Authorizer, Bộ nhớ đệm phản hồi (Response Caching) và Tường lửa tầng ứng dụng (AWS WAF).