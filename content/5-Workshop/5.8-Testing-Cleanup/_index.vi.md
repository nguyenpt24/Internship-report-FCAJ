---
title: "Kiểm thử Real-Time & Cleanup"
date: 2026-08-04
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

### 5.8. Kiểm Thử Độc Lập Trình Duyệt Real-Time & Kịch Bản Cleanup Tài Nguyên

#### Mục tiêu bài lab
Kiểm thử tính năng gửi/nhận tin nhắn thời gian thực giữa 2 tab trình duyệt độc lập và thực thi kịch bản AWS CLI cleanup để dọn dẹp các tài nguyên sau khi hoàn thành.

---

#### 1. Kiểm thử ứng dụng Chat Real-Time trên 2 tab trình duyệt
1. Mở đường dẫn CloudFront Domain URL (hoặc mở tệp `index.html`) trên **Tab trình duyệt 1 (User A)**.
2. Mở đường dẫn trên ở một **Tab ẩn danh hoặc Trình duyệt 2 (User B)**.
3. Kiểm tra trạng thái hiển thị: Cả 2 tab đều báo `Connected Online (WebSocket Active)`.
4. Nhập tin nhắn trên Tab 1 và nhấn **Gửi**.
5. **Xác nhận**: Tin nhắn xuất hiện gần như tức thì (< 100ms) trên cả Tab 1 và Tab 2.

![Màn hình kiểm thử Chat Real-Time thành công song song trên 2 tab trình duyệt](/images/5.8-testing-chat-two-tabs.png)

---

#### 2. Kịch bản dọn dẹp tài nguyên (Cleanup Scripts)
Để tránh phát sinh chi phí ngoài ý muốn sau khi kết thúc bài workshop, chạy các lệnh AWS CLI sau trên Terminal để xóa tài nguyên:

```bash
# 1. Xóa đường dẫn WebSocket API Gateway
aws apigatewayv2 delete-api --api-id <YOUR-API-ID>

# 2. Xóa các hàm AWS Lambda
aws lambda delete-function --function-name onConnect
aws lambda delete-function --function-name onDisconnect
aws lambda delete-function --function-name sendMessage

# 3. Xóa bảng Amazon DynamoDB
aws dynamodb delete-table --table-name WebSocketConnections

# 4. Xóa S3 Bucket Frontend
aws s3 rb s3://my-serverless-chat-frontend-2026 --force

# 5. Gỡ bỏ các Policy và xóa IAM Role ChatAppLambdaRole
aws iam detach-role-policy --role-name ChatAppLambdaRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam detach-role-policy --role-name ChatAppLambdaRole --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess
aws iam detach-role-policy --role-name ChatAppLambdaRole --policy-arn arn:aws:iam::aws:policy/AmazonAPIGatewayInvokeFullAccess
aws iam delete-role-policy --role-name ChatAppLambdaRole --policy-name WebSocketManageConnectionsPolicy
aws iam delete-role --role-name ChatAppLambdaRole
```

![Kết quả thực thi lệnh AWS CLI dọn dẹp các dịch vụ API Gateway, Lambda và DynamoDB](/images/5.8-cleanup-aws-cli-result(1).png)

![Kết quả thực thi lệnh AWS CLI dọn dẹp S3 Bucket và IAM Role](/images/5.8-cleanup-aws-cli-result(2).png)
