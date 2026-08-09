---
title: "Lập trình AWS Lambda Backend"
date: 2026-08-04
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

### 5.4. Lập trình Mã Nguồn AWS Lambda Handlers (Node.js)

#### Mục tiêu bài lab

Tạo 3 hàm AWS Lambda Node.js (`onConnect`, `onDisconnect`, `sendMessage`) để xử lý các sự kiện của giao thức WebSocket, đồng thời gán IAM Role `ChatAppLambdaRole` đã khởi tạo từ bài lab 5.2.

---

#### Quy trình tạo hàm AWS Lambda trên AWS Management Console:

1. Mở dịch vụ **AWS Lambda Console** ➔ Nhấn **Create function**.
2. Chọn tùy chọn **Author from scratch**.
3. Cấu hình thông tin cơ bản:
   - **Runtime**: Chọn **Node.js 22.x** (hoặc **Node.js 20.x**).
   - **Architecture**: Giữ mặc định **x86_64**.
4. Cấu hình quyền thực thi (dưới mục **Change default execution role**):
   - Chọn **Use an existing role**.
   - Trong danh sách **Existing role**, chọn `ChatAppLambdaRole` (Role đã được gán các quyền DynamoDB, CloudWatch và API Gateway ở Bài lab 5.2).
5. Nhấn **Create function**.
6. Tại tab **Code** (mục **Code source**):
   - Trong cây thư mục bên trái, nhấp chuột phải vào tệp `index.mjs` ➔ chọn **Rename** ➔ đổi tên thành **`index.js`** (hoặc `index.cjs`) để hỗ trợ cú pháp `require(...)`.
   - Dán mã nguồn tương ứng và nhấn nút **Deploy** để hoàn tất triển khai.

---

#### 1. Hàm Lambda `onConnect`

Hàm tiếp nhận kết nối WebSocket mới và lưu `connectionId` vào bảng DynamoDB `WebSocketConnections`.

##### Thao tác khởi tạo:

- **Function name**: `onConnect`
- **Existing role**: `ChatAppLambdaRole`

```javascript
const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
const { DynamoDBDocumentClient, PutCommand } = require("@aws-sdk/lib-dynamodb");

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

exports.handler = async (event) => {
  const connectionId = event.requestContext.connectionId;
  const params = {
    TableName: "WebSocketConnections",
    Item: { connectionId: connectionId },
  };

  try {
    await docClient.send(new PutCommand(params));
    return { statusCode: 200, body: "Connected." };
  } catch (err) {
    return {
      statusCode: 500,
      body: "Failed to connect: " + JSON.stringify(err),
    };
  }
};
```

![Giao diện cấu hình hàm Lambda onConnect trên AWS Console](/images/5.4-lambda-onconnect-console.png)

---

#### 2. Hàm Lambda `onDisconnect`

Hàm xóa `connectionId` khỏi bảng DynamoDB khi client ngắt kết nối WebSocket.

##### Thao tác khởi tạo:

- **Function name**: `onDisconnect`
- **Existing role**: `ChatAppLambdaRole`

```javascript
const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
const {
  DynamoDBDocumentClient,
  DeleteCommand,
} = require("@aws-sdk/lib-dynamodb");

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

exports.handler = async (event) => {
  const connectionId = event.requestContext.connectionId;
  const params = {
    TableName: "WebSocketConnections",
    Key: { connectionId: connectionId },
  };

  try {
    await docClient.send(new DeleteCommand(params));
    return { statusCode: 200, body: "Disconnected." };
  } catch (err) {
    return {
      statusCode: 500,
      body: "Failed to disconnect: " + JSON.stringify(err),
    };
  }
};
```

![Giao diện cấu hình hàm Lambda onDisconnect trên AWS Console](/images/5.4-lambda-ondisconnect-console.png)

---

#### 3. Hàm Lambda `sendMessage`

Hàm phát tin nhắn tới tất cả client đang kết nối và tự động xóa kết nối ngắt đột ngột (lỗi `410 Gone`).

##### Thao tác khởi tạo:

- **Function name**: `sendMessage`
- **Existing role**: `ChatAppLambdaRole`

```javascript
const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
const {
  DynamoDBDocumentClient,
  ScanCommand,
  DeleteCommand,
} = require("@aws-sdk/lib-dynamodb");
const {
  ApiGatewayManagementApiClient,
  PostToConnectionCommand,
} = require("@aws-sdk/client-apigatewaymanagementapi");

const dbClient = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(dbClient);

exports.handler = async (event) => {
  const domainName = event.requestContext.domainName;
  const stage = event.requestContext.stage;
  const endpoint = `https://${domainName}/${stage}`;
  const apiGateway = new ApiGatewayManagementApiClient({ endpoint: endpoint });

  let postData;
  try {
    const body =
      typeof event.body === "string" ? JSON.parse(event.body) : event.body;
    postData = body.data;
  } catch (e) {
    return { statusCode: 400, body: "Invalid payload format." };
  }

  const connections = await docClient.send(
    new ScanCommand({ TableName: "WebSocketConnections" }),
  );

  const sendMessages = connections.Items.map(async ({ connectionId }) => {
    try {
      await apiGateway.send(
        new PostToConnectionCommand({
          ConnectionId: connectionId,
          Data: Buffer.from(
            JSON.stringify({
              message: postData,
              sender: event.requestContext.connectionId,
            }),
          ),
        }),
      );
    } catch (e) {
      if (e.$metadata && e.$metadata.httpStatusCode === 410) {
        // Connection staled, remove from DynamoDB
        await docClient.send(
          new DeleteCommand({
            TableName: "WebSocketConnections",
            Key: { connectionId },
          }),
        );
      }
    }
  });

  await Promise.all(sendMessages);
  return { statusCode: 200, body: "Data sent." };
};
```

![Giao diện cấu hình hàm Lambda sendMessage và gán IAM Role ChatAppLambdaRole](/images/5.4-lambda-sendmessage-role.png)
