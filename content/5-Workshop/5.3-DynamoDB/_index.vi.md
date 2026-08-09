---
title: "Khởi tạo Amazon DynamoDB"
date: 2026-08-04
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

### 5.3. Khởi tạo Bảng Amazon DynamoDB (WebSocketConnections)

#### Mục tiêu bài lab

Tạo bảng NoSQL `WebSocketConnections` trên Amazon DynamoDB để lưu trữ các ID kết nối WebSocket active (`connectionId`) với độ trễ phản hồi tính bằng mili-giây.

---

#### Hướng dẫn thực hiện

##### Thao tác trên AWS Management Console:

1. Mở dịch vụ **Amazon DynamoDB Console**.
2. Nhấn **Create table**.
3. Cấu hình bảng:
   - **Table name**: `WebSocketConnections`
   - **Partition key**: `connectionId` (Data type: **String**)
   - **Table class**: DynamoDB Standard
   - **Read/write capacity settings**: Select **On-demand (Pay-per-request)** để tối ưu chi phí trong Free Tier.

![Màn hình nhập thông tin Table Name và Partition Key trong DynamoDB Console](/images/5.3-dynamodb-create-table.png)

4. Nhấn **Create table** và chờ khoảng vài giây để bảng chuyển sang trạng thái `Active`.

![Bảng WebSocketConnections ở trạng thái Active thành công](/images/5.3-dynamodb-active-status.png)

---

##### Hoặc khởi tạo nhanh bằng AWS CLI:

```bash
aws dynamodb create-table \
    --table-name WebSocketConnections \
    --attribute-definitions AttributeName=connectionId,AttributeType=S \
    --key-schema AttributeName=connectionId,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```
