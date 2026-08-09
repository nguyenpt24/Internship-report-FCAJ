---
title: "Configure WebSocket API Gateway"
date: 2026-08-04
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

### 5.5. Create & Configure Amazon API Gateway (WebSocket API)

#### Module Goal
Provision a **WebSocket API** on Amazon API Gateway, bind lifecycle routes (`$connect`, `$disconnect`, `sendmessage`) to Lambda handlers, and deploy the `production` stage.

---

#### Step-by-Step Instructions

##### Step 1: Create WebSocket API
1. Open the **Amazon API Gateway Console**.
2. Click **Build** under **WebSocket API**.
3. Specify API configuration:
   - **API name**: `RealtimeChatWebSocketAPI`
   - **Route selection expression**: `$request.body.action`
   - **IP address type**: Keep default **IPv4**.

![Creating WebSocket API with Route selection expression](/images/5.5-apigateway-websocket-create.png)

---

##### Step 2: Define Event Routes
1. Under Predefined routes: Select **$connect** and **$disconnect**.
2. Under Custom routes: Add route key `sendmessage`.

![Configuring $connect, $disconnect, and sendmessage routes](/images/5.5-apigateway-routes-config.png)

---

##### Step 3: Attach AWS Lambda Integrations
1. Attach Route **$connect** ➔ Lambda function `onConnect`.
2. Attach Route **$disconnect** ➔ Lambda function `onDisconnect`.
3. Attach Route **sendmessage** ➔ Lambda function `sendMessage`.

![Mapping routes to corresponding Lambda handler functions](/images/5.5-apigateway-lambda-integration.png)

---

##### Step 4: Deploy Stage and Retrieve WSS URL
1. Specify Stage Name: `production` ➔ Click **Next** and **Create and Deploy**.
2. Copy the generated **WebSocket URL** (`wss://<api-id>.execute-api.us-east-1.amazonaws.com/production`).

![WebSocket Endpoint WSS URL displayed on API Gateway Console](/images/5.5-apigateway-wss-endpoint.png)
