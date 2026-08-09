---
title: "Worklog Tuần 2"
date: 2026-06-15
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

- Nắm vững quản lý truy cập và phân quyền bảo mật với AWS IAM.
- Hiểu kiến trúc mạng riêng ảo Amazon VPC, Subnet, Route Table và Security Group.
- Thực hành khởi tạo máy chủ EC2, quản lý lưu trữ EBS và hosting website tĩnh với Amazon S3.
- Khởi tạo tài nguyên nền tảng cho dự án **Serverless Real-Time Chat App** (IAM Role & S3 Bucket).

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                                                                                                                           | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Tìm hiểu & thực hành IAM: IAM User, Group, Policy <br> - Khởi tạo IAM Role `ChatAppLambdaRole` cấp quyền ghi CloudWatch & thao tác DynamoDB cho dự án Chat        | 15/06/2026   | 15/06/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - Tìm hiểu kiến thức về mạng với Amazon Virtual Private Cloud (VPC) <br> - Cấu hình Public/Private Subnet, Internet Gateway, Route Tables cho môi trường phát triển | 16/06/2026   | 16/06/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Khởi tạo máy chủ ảo Amazon EC2, chọn Instance Types, AMI, gắn EBS Volume <br> - Thực hành remote SSH an toàn vào EC2                                              | 17/06/2026   | 17/06/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - Tìm hiểu & phát triển ứng dụng trên đám mây với AWS Cloud9 <br> - Cài đặt Node.js và AWS CLI môi trường lập trình dự án Chat                                      | 18/06/2026   | 18/06/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | - **Thực hành:** Khởi tạo Amazon S3 Bucket `my-serverless-chat-frontend-2026` <br> - Cấu hình Static Website Hosting chuẩn bị lưu trữ giao diện Web Chat Client     | 19/06/2026   | 19/06/2026      | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 2:

- Khởi tạo thành công IAM Role `ChatAppLambdaRole` với đầy đủ quyền ghi CloudWatch Logs và thao tác DynamoDB chuẩn bị cho backend ứng dụng Chat.

- Xây dựng thành công mạng ảo VPC hoàn chỉnh cho môi trường lab phát triển.

- Khởi tạo S3 Bucket `my-serverless-chat-frontend-2026` và bật tính năng Static Website Hosting để sẵn sàng lưu trữ Web Chat Frontend.

- Thành thạo công cụ AWS Cloud9 và AWS CLI phục vụ quá trình viết code ứng dụng.
