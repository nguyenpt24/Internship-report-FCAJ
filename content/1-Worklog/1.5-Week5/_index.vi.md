---
title: "Worklog Tuần 5"
date: 2026-07-06
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

- Tìm hiểu quy trình và công cụ dịch chuyển máy ảo, cơ sở dữ liệu và ứng dụng lên AWS.
- Thực hành tự động hóa hạ tầng dưới dạng mã (Infrastructure as Code - IaC) với AWS CloudFormation và AWS CDK.
- Viết thử nghiệm template CloudFormation / AWS CDK định nghĩa hạ tầng dự án **Serverless Real-Time Chat App**.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu dịch chuyển máy ảo với AWS VM Import/Export <br> - Giải pháp phục hồi sau thảm họa với AWS Elastic Disaster Recovery (DRS) | 06/07/2026 | 06/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Tìm hiểu dịch chuyển cơ sở dữ liệu với AWS Database Migration Service (DMS) <br> - Sử dụng Schema Conversion Tool (SCT) để chuyển đổi schema DB | 07/07/2026 | 07/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Khởi tạo Hạ tầng dưới dạng mã với AWS CloudFormation <br> - **Viết Template IaC cho dự án Chat**: Khai báo tự động DynamoDB Table & S3 Bucket Chat Frontend | 08/07/2026 | 08/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Bộ công cụ phát triển đám mây AWS CDK (cơ bản & nâng cao) <br> - Thử nghiệm đóng gói hạ tầng bằng TypeScript với AWS CDK | 09/07/2026 | 09/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Thực hành:** Workshop Infrastructure as Code (IaC) <br> - Cấu hình tự động hóa snapshot với Amazon EBS Data Lifecycle Manager | 10/07/2026 | 10/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 5:

- Xây dựng thành công mẫu template CloudFormation / AWS CDK giúp tự động hóa việc khởi tạo bảng DynamoDB và S3 Bucket cho ứng dụng Chat.

- Thành thạo phương pháp quản lý tài nguyên dưới dạng mã (IaC), dễ dàng nhân bản môi trường phát triển & kiểm thử.
