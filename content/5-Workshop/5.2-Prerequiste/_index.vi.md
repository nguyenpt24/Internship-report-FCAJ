---
title: "Chuẩn bị môi trường & IAM Role"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

### 5.2. Chuẩn bị môi trường & Khởi tạo IAM Role cho Lambda

#### Mục tiêu bài lab
Tạo IAM Role `ChatAppLambdaRole` cấp quyền cho AWS Lambda ghi log vào CloudWatch Logs, thao tác dữ liệu với DynamoDB và phát tin nhắn qua API Gateway WebSocket.

---

#### Hướng dẫn thực hiện

##### Bước 1: Kiểm tra cấu hình AWS CLI
Mở Terminal trên máy tính và kiểm tra kết nối với tài khoản AWS:

```bash
aws sts get-caller-identity
```

![Kết quả chạy lệnh aws sts get-caller-identity xác nhận tài khoản AWS](/images/5.2-aws-cli-identity.png)

---

##### Bước 2: Tạo IAM Role trên AWS Management Console
1. Truy cập vào dịch vụ **IAM Management Console**.
2. Chọn **Roles** ➔ Nhấn **Create role**.
3. Chọn **AWS service**, dưới mục Use case chọn **Lambda** ➔ Nhấn **Next**.

![Giao diện chọn trusted entity type cho IAM Role Lambda](/images/5.2-iam-role-trusted-entity.png)

4. Tại bước **Add permissions**, tìm và tích chọn các Policy sau:
   - `AWSLambdaBasicExecutionRole` (Quyền ghi log CloudWatch)
   - `AmazonDynamoDBFullAccess` (Quyền đọc/ghi bảng DynamoDB)
   - `AmazonAPIGatewayInvokeFullAccess` (Quyền gọi API Gateway)

![Giao diện gán quyền Permissions Policies cho IAM Role](/images/5.2-iam-role-policies.png)

5. Đặt tên Role: `ChatAppLambdaRole` ➔ Nhấn **Create role**.

6. **Thêm Inline Policy cấp quyền quản lý kết nối WebSocket (`execute-api:ManageConnections`)**:
   - Chọn Role `ChatAppLambdaRole` vừa tạo.
   - Tại tab **Permissions**, chọn **Add permissions** ➔ **Create inline policy**.
   - Chọn tab **JSON** và dán chính sách sau:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": [
                     "execute-api:ManageConnections"
                 ],
                 "Resource": "arn:aws:execute-api:*:*:*/*/*/@connections/*"
             }
         ]
     }
     ```
   - Nhập tên Policy `WebSocketManageConnectionsPolicy` ➔ Nhấn **Create policy**.

![Màn hình tạo Inline Policy WebSocketManageConnectionsPolicy thành công](/images/5.2-iam-inline-policy.png)