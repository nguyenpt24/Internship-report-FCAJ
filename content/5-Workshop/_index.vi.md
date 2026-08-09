---
title: "Workshop"
date: 2026-08-04
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Workshop: Triển Khai Ứng Dụng Chat Thời Gian Thực Serverless Trực Tiếp Trên AWS
## (Serverless Real-Time Chat Application on AWS)

---

### Giới thiệu bài Workshop

Bài Workshop này hướng dẫn từng bước chi tiết cách xây dựng và triển khai một **Ứng dụng Chat thời gian thực không máy chủ (Serverless Real-Time Chat App)** hoàn chỉnh trên đám mây AWS.

Ứng dụng kết hợp giao thức **WebSocket (API Gateway)** với mô hình xử lý hướng sự kiện (**AWS Lambda**), lưu trữ dữ liệu NoSQL tốc độ cao (**Amazon DynamoDB**), hosting giao diện web (**Amazon S3 + CloudFront CDN**), quản lý xác thực người dùng (**Amazon Cognito**) và giám sát tập trung (**Amazon CloudWatch**).

---

### Danh sách các bài lab trong Workshop:

1. **[5.1. Tổng quan kiến trúc](5.1-workshop-overview/)**: Khái niệm WebSocket Serverless & Sơ đồ luồng dữ liệu.
2. **[5.2. Chuẩn bị môi trường & IAM Role](5.2-prerequiste/)**: Cấu hình AWS CLI & Khởi tạo IAM Role cho Lambda.
3. **[5.3. Khởi tạo Amazon DynamoDB](5.3-dynamodb/)**: Tạo bảng `WebSocketConnections` lưu trữ ID kết nối active.
4. **[5.4. Lập trình AWS Lambda Backend](5.4-lambda-backend/)**: Viết mã nguồn Node.js cho `onConnect`, `onDisconnect` và `sendMessage`.
5. **[5.5. Cấu hình WebSocket API Gateway](5.5-api-gateway/)**: Định tuyến route `$connect`, `$disconnect`, `sendmessage` & deploy Stage `production`.
6. **[5.6. Triển khai Web Frontend & CloudFront](5.6-frontend-s3/)**: Hosting trang Chat Web (`index.html`) trên S3 và phân phối qua CloudFront.
7. **[5.7. Tích hợp Cognito & CloudWatch Alarms](5.7-cognito-cloudwatch/)**: Cấu hình xác thực người dùng & Giám sát báo lỗi tự động.
8. **[5.8. Kiểm thử Real-Time & Cleanup](5.8-testing-cleanup/)**: Thử nghiệm chat trên 2 tab trình duyệt & Kịch bản dọn dẹp tài nguyên.