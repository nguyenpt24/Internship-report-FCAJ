---
title: "Worklog Tuần 7"
date: 2026-07-20
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

- Cấu hình giải pháp sao lưu dữ liệu tự động cho bảng DynamoDB `WebSocketConnections` với AWS Backup.
- Nghiên cứu cơ chế nhắn tin bất đồng bộ với SQS/SNS áp dụng cho việc truyền tải thông báo trong Chat App.
- Thiết lập kết nối mạng riêng giữa các VPC và cụm máy chủ chịu lỗi cao (High Availability).

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu & cấu hình bảo vệ dữ liệu tự động với AWS Backup <br> - **Lập lịch sao lưu DynamoDB**: Cấu hình quy tắc sao lưu định kỳ cho bảng `WebSocketConnections` | 20/07/2026 | 20/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Tích hợp mạng nâng cao với VPC Peering <br> - Quản lý mạng tập trung quy mô lớn với AWS Transit Gateway | 21/07/2026 | 21/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Xây dựng hệ thống nhắn tin ứng dụng với Amazon SQS và SNS <br> - Mô hình Pub/Sub thông báo tin nhắn và hàng đợi xử lý bất đồng bộ trong kiến trúc Chat | 22/07/2026 | 22/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Chia sẻ dữ liệu lưu trữ khối với Amazon EBS Multi-Attach <br> - Tính sẵn sàng cao cho cơ sở dữ liệu với EBS Multi-Attach và Systems Manager | 23/07/2026 | 23/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:** Cụm chịu lỗi Windows Server trên AWS <br> - Cấu hình SQL Server tính sẵn sàng cao trên AWS (2019 / 2022) | 24/07/2026 | 24/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 7:

- Cấu hình thành công AWS Backup tự động sao lưu bảng dữ liệu DynamoDB của dự án Chat.

- Hiểu mô hình kiến trúc Pub/Sub (Amazon SNS) và hàng đợi (Amazon SQS) để mở rộng hệ thống xử lý tin nhắn quy mô lớn.

- Nắm vững giải pháp thiết kế hệ thống chịu lỗi cao High Availability trên AWS.
