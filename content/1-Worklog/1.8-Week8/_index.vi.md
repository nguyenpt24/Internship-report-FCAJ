---
title: "Worklog Tuần 8"
date: 2026-07-27
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:

- Sử dụng Docker đóng gói ứng dụng và điều phối container trên Amazon ECS Fargate.
- Tự động hóa luồng triển khai CI/CD code ứng dụng với AWS CodePipeline.
- Phân tích và dự báo chi phí dự án **Serverless Real-Time Chat App** với AWS Budgets, AWS Glue & Athena.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Sử dụng container với Docker, tạo Dockerfile & chạy container cục bộ <br> - Điều phối container đám mây với Amazon ECS và AWS ECR | 27/07/2026 | 27/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Định nghĩa hạ tầng dưới dạng mã cho ECS với AWS CDK <br> - Triển khai serverless container với Amazon ECS Fargate | 28/07/2026 | 28/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Xây dựng Pipeline CI/CD với AWS CodePipeline <br> - **Thiết lập CI/CD**: Tự động hóa triển khai code mã nguồn Lambda của Chat App từ GitHub | 29/07/2026 | 29/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - Tìm hiểu dịch vụ lưu trữ tập tin Windows với Amazon FSx <br> - Triển khai lưu trữ lai với AWS Storage Gateway | 30/07/2026 | 30/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - **Tối ưu chi phí Dự án Chat:** Cấu hình AWS Budgets mốc $1.00 <br> - **Phân tích chi phí:** Truy vấn dữ liệu chi phí tài nguyên dự án Chat với AWS Glue & Amazon Athena | 31/07/2026 | 31/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 8:

- Thiết lập thành công AWS CodePipeline tự động cập nhật code cho các hàm Lambda dự án Chat mỗi khi có commit mới.

- Cấu hình cảnh báo chi phí AWS Budgets mốc $1.00 đảm bảo dự án nằm hoàn toàn trong phạm vi gói AWS Free Tier.

- Nắm vững công cụ phân tích dữ liệu chi phí AWS Glue & Amazon Athena.
