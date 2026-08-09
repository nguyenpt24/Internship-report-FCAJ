---
title: "Worklog Tuần 3"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

- Nắm kiến thức về cơ sở dữ liệu quan hệ Amazon RDS và cơ sở dữ liệu NoSQL Amazon DynamoDB.
- Hiểu giải pháp bộ nhớ đệm ElastiCache và quản lý tên miền Amazon Route 53.
- Khởi tạo bảng DynamoDB `WebSocketConnections` lưu trữ kết nối active cho ứng dụng **Serverless Real-Time Chat App**.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu & khởi tạo Amazon Relational Database Service (RDS) <br> - Cấu hình Multi-AZ, Read Replica và kết nối từ máy chủ EC2 | 22/06/2026 | 22/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Tìm hiểu cơ sở dữ liệu NoSQL với Amazon DynamoDB <br> - **Thực hành tạo Bảng `WebSocketConnections`**: Cấu hình Primary Key `connectionId` (String), Pay-per-request billing | 23/06/2026 | 23/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Thao tác với bộ nhớ đệm ElastiCache (Redis/Memcached) <br> - Thực hành các lệnh AWS CLI truy vấn thử nghiệm dữ liệu trên bảng DynamoDB | 24/06/2026 | 24/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Quản lý DNS lai và định tuyến tên miền với Amazon Route 53 <br> - Tạo Hosted Zone, cấu hình Record A, CNAME trỏ về EC2 / S3 | 25/06/2026 | 25/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:** Mở rộng quy mô ứng dụng với EC2 Auto Scaling <br> - Kết nối Auto Scaling Group với Load Balancer & cài đặt giám sát Amazon CloudWatch | 26/06/2026 | 26/06/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 3:

- Khởi tạo thành công bảng DynamoDB `WebSocketConnections` với `connectionId` làm Partition Key, sẵn sàng cho việc lưu trữ ID kết nối WebSocket của ứng dụng Chat.

- Thành thạo thao tác CRUD dữ liệu trên DynamoDB bằng cả giao diện Console lẫn AWS CLI SDK.

- Hiểu và ứng dụng mô hình lưu trữ NoSQL siêu tốc độ cho dự án Chat thời gian thực.
