---
title: "Author AWS Lambda Handlers"
date: 2026-08-04
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

### 5.4. Author AWS Lambda Event Handlers (Node.js)

#### Module Goal

Create 3 Node.js AWS Lambda functions (`onConnect`, `onDisconnect`, `sendMessage`) to handle WebSocket lifecycle events, attaching the execution IAM Role `ChatAppLambdaRole` created in lab module 5.2.

---

#### Step-by-Step Procedure to Create AWS Lambda Functions via AWS Console:

1. Open the **AWS Lambda Console** ➔ Click **Create function**.
2. Select **Author from scratch**.
3. Basic Information Configuration:
   - **Runtime**: Select **Node.js 22.x** (or **Node.js 20.x**).
   - **Architecture**: Keep default **x86_64**.
4. Change execution role permissions (under **Change default execution role**):
   - Select **Use an existing role**.
   - Under **Existing role**, choose `ChatAppLambdaRole` (attached with DynamoDB, CloudWatch, and API Gateway policies in Lab 5.2).
5. Click **Create function**.
6. Under the **Code** tab (**Code source** panel):
   - In the file tree on the left, right-click `index.mjs` ➔ select **Rename** ➔ rename it to **`index.js`** (or `index.cjs`) to support CommonJS `require(...)` syntax.
   - Paste the handler source code and click **Deploy** to publish your changes.

---

#### 1. `onConnect` Lambda Handler

Handles incoming WebSocket connection requests and saves `connectionId` into the DynamoDB `WebSocketConnections` table.

##### Creation Specs:

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

![Configuration screen for onConnect Lambda function on AWS Console](/images/5.4-lambda-onconnect-console.png)

---

#### 2. `onDisconnect` Lambda Handler

Removes `connectionId` from DynamoDB when a WebSocket client disconnects.

##### Creation Specs:

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

![Configuration screen for onDisconnect Lambda function on AWS Console](/images/5.4-lambda-ondisconnect-console.png)

---

#### 3. `sendMessage` Lambda Handler

Broadcasts chat messages to all connected clients and purges stale connections (HTTP `410 Gone`).

##### Creation Specs:

- **Function name**: `sendMessage`
- **Existing role**: `ChatAppLambdaRole`

```javascript
const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
const {
  DynamoDBDocumentClient,
  ScanCommand,
  PutCommand,
  QueryCommand,
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
  const connectionId = event.requestContext.connectionId;
  const endpoint = `https://${domainName}/${stage}`;
  const apiGateway = new ApiGatewayManagementApiClient({ endpoint: endpoint });

  let rawData;
  let payloadObj;
  try {
    const body = typeof event.body === "string" ? JSON.parse(event.body) : event.body;
    rawData = body.data;
    payloadObj = typeof rawData === "string" ? JSON.parse(rawData) : rawData;
  } catch (e) {
    payloadObj = { type: "chat", text: String(rawData || ""), roomId: "general" };
  }

  const roomId = payloadObj.roomId || "general";

  // 1. Handle Get History Action
  if (payloadObj.type === "gethistory") {
    try {
      const historyResult = await docClient.send(
        new QueryCommand({
          TableName: "ChatMessages",
          KeyConditionExpression: "roomId = :r",
          ExpressionAttributeValues: { ":r": roomId },
          ScanIndexForward: true, // Chronological order
          Limit: 50
        })
      );

      await apiGateway.send(
        new PostToConnectionCommand({
          ConnectionId: connectionId,
          Data: Buffer.from(
            JSON.stringify({
              message: JSON.stringify({
                type: "history",
                roomId: roomId,
                history: historyResult.Items || []
              })
            })
          )
        })
      );
      return { statusCode: 200, body: "History sent." };
    } catch (err) {
      console.error("Error fetching history:", err);
      return { statusCode: 500, body: "Failed to fetch history." };
    }
  }

  // 2. Persist chat message to DynamoDB ChatMessages table
  if (payloadObj.type === "chat" && (payloadObj.text || payloadObj.image)) {
    const messageItem = {
      roomId: roomId,
      timestamp: payloadObj.timestamp || new Date().toISOString(),
      messageId: "msg_" + Math.random().toString(36).substring(2, 10),
      username: payloadObj.username || "User",
      text: payloadObj.text || "",
      image: payloadObj.image || null,
      clientId: payloadObj.clientId || "unknown"
    };

    try {
      await docClient.send(
        new PutCommand({
          TableName: "ChatMessages",
          Item: messageItem
        })
      );
    } catch (err) {
      console.error("Error saving chat message:", err);
    }
  }

  // 3. Broadcast payload to connected WebSocket clients
  const connections = await docClient.send(
    new ScanCommand({ TableName: "WebSocketConnections" })
  );

  const sendMessages = connections.Items.map(async ({ connectionId: targetId }) => {
    try {
      await apiGateway.send(
        new PostToConnectionCommand({
          ConnectionId: targetId,
          Data: Buffer.from(
            JSON.stringify({
              message: rawData,
              sender: connectionId
            })
          )
        })
      );
    } catch (e) {
      if (e.$metadata && e.$metadata.httpStatusCode === 410) {
        await docClient.send(
          new DeleteCommand({
            TableName: "WebSocketConnections",
            Key: { connectionId: targetId }
          })
        );
      }
    }
  });

  await Promise.all(sendMessages);
  return { statusCode: 200, body: "Data processed." };
};
```

![Assigning ChatAppLambdaRole to sendMessage Lambda function](/images/5.4-lambda-sendmessage-role.png)
