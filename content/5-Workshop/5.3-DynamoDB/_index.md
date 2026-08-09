---
title: "Create Amazon DynamoDB Table"
date: 2026-08-04
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

### 5.3. Create Amazon DynamoDB Table (WebSocketConnections)

#### Module Goal

Provision the Amazon DynamoDB NoSQL table `WebSocketConnections` to store active WebSocket client connection IDs (`connectionId`) with single-digit millisecond latency.

---

#### Step-by-Step Instructions

##### Via AWS Management Console:

1. Open the **Amazon DynamoDB Console**.
2. Click **Create table**.
3. Configure Table Settings:
   - **Table name**: `WebSocketConnections`
   - **Partition key**: `connectionId` (Data type: **String**)
   - **Table class**: DynamoDB Standard
   - **Read/write capacity settings**: Select **On-demand (Pay-per-request)** to operate within Free Tier bounds.

![Entering Table Name and Partition Key in DynamoDB Console](/images/5.3-dynamodb-create-table.png)

4. Click **Create table** and wait a few seconds until the status turns `Active`.

![WebSocketConnections table in Active status](/images/5.3-dynamodb-active-status.png)

---

##### Or Provision via AWS CLI:

```bash
aws dynamodb create-table \
    --table-name WebSocketConnections \
    --attribute-definitions AttributeName=connectionId,AttributeType=S \
    --key-schema AttributeName=connectionId,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST
```
