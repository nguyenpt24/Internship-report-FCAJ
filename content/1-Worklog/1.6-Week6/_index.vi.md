---
title: "Worklog Tuần 6"
date: 2026-07-13
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6:

- Quản lý vận hành máy chủ từ xa an toàn không cần mở cổng SSH/RDP với AWS Systems Manager.
- Xây dựng hệ thống giám sát nâng cao với CloudWatch, Grafana và VPC Flow Logs.
- Khởi tạo Amazon Cognito User Pool để xác thực người dùng đăng nhập ứng dụng **Serverless Real-Time Chat App**.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Quản lý hệ thống với AWS Systems Manager (SSM) <br> - Truy cập máy chủ từ xa an toàn với Systems Manager Session Manager | 13/07/2026 | 13/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Giám sát nâng cao với CloudWatch và Grafana <br> - Workshop CloudWatch Nâng cao, cấu hình VPC Flow Logs & Tags | 14/07/2026 | 14/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Kiểm soát quyền truy cập an toàn với IAM Permission Boundaries <br> - Cấu hình nâng cao IAM Policies cho Lambda dự án Chat & VPC Endpoints | 15/07/2026 | 15/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Bảo vệ ứng dụng web với AWS WAF và quản lý mã hóa với AWS KMS <br> - Quản lý thông tin xác thực an toàn với AWS Secrets Manager | 16/07/2026 | 16/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành Bảo mật:** Khởi tạo Amazon Cognito User Pool & App Client cho ứng dụng Chat <br> - Cấu hình JWT Token verification cấp quyền truy cập ứng dụng Chat | 17/07/2026 | 17/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 6:

- Khởi tạo thành công Amazon Cognito User Pool & App Client phục vụ quản lý đăng ký, đăng nhập và cấp JWT Token cho người dùng Chat App.

- Cấu hình IAM Least Privilege Policy cho các hàm Lambda chỉ được phép truy xuất đúng bảng DynamoDB `WebSocketConnections`.

- Làm chủ công cụ quản trị an toàn AWS Systems Manager và bảo mật mã hóa KMS.
