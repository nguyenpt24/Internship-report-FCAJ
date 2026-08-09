---
title: "Worklog Tuần 4"
date: 2026-06-29
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

- Tối ưu hóa phân phối nội dung toàn cầu với Amazon CloudFront và điện toán biên CloudFront & Lambda@Edge.
- Đơn giản hóa điện toán ứng dụng với Amazon Lightsail.
- Cấu hình CloudFront Distribution trỏ về S3 Bucket `my-serverless-chat-frontend-2026` để phân phối ứng dụng Chat qua HTTPS.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu & cấu hình mạng phân phối nội dung Amazon CloudFront <br> - **Tạo CloudFront Distribution**: Cấu hình Origin trỏ về S3 Bucket Chat Frontend | 29/06/2026 | 29/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 3 | - Tìm hiểu điện toán biên với CloudFront và Lambda@Edge <br> - Thử nghiệm hàm Lambda@Edge kiểm tra header an toàn cho trang web Chat | 30/06/2026 | 30/06/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 4 | - Đơn giản hóa điện toán với dịch vụ Amazon Lightsail <br> - Khởi tạo Lightsail Instance, cấu hình firewall và cài đặt web app đơn giản | 01/07/2026 | 01/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - **Thực hành:** Triển khai container ứng dụng với Amazon Lightsail Containers | 02/07/2026 | 02/07/2026 | <https://cloudjourney.awsstudygroup.com/> |
| 6 | - Tìm hiểu giải pháp Ứng dụng Windows trên AWS <br> - Cấu hình dịch vụ thư mục với AWS Managed Microsoft AD và kết nối máy chủ Windows EC2 | 03/07/2026 | 03/07/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 4:

- Tạo thành công CloudFront Distribution phục vụ phân phối Web Chat Frontend tốc độ cao, đảm bảo kết nối bảo mật qua chuẩn HTTPS.

- Hiểu cách ứng dụng Lambda@Edge tại Edge Location để tùy biến header bảo mật cho website.
