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

![Assigning ChatAppLambdaRole to sendMessage Lambda function](/images/5.4-lambda-sendmessage-role.png)
