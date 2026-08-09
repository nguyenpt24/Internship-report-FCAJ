---
title: "Architecture Overview"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### 5.1. Project & Serverless WebSocket Architecture Overview

#### Module Introduction
In this hands-on lab, we explore bi-directional communication via the **WebSocket** protocol and why an **AWS Serverless architecture** provides the ideal foundation for building real-time chat applications.

---

#### Data Flow Architecture Model

![Overall AWS Serverless Real-Time Chat Architecture Diagram](/images/serverless-chat-architecture.png)

1. **Client Connection**: A user opens the Web Chat Frontend interface. The browser initiates a WebSocket WSS handshake (`wss://...execute-api.us-east-1.amazonaws.com/production`).
2. **API Gateway Handshake**: Amazon API Gateway handles the WebSocket connection request and triggers the `onConnect` Lambda function.
3. **Save Connection ID**: The `onConnect` function extracts the assigned `connectionId` and persists it into the Amazon DynamoDB table `WebSocketConnections`.
4. **Message Transmission**: When a user submits a message, the client sends a JSON payload: `{"action": "sendmessage", "data": "Hello World!"}`.
5. **Broadcasting Messages**: API Gateway routes the request to the `sendMessage` Lambda function. The function queries all active `connectionId` records in DynamoDB and invokes the API Gateway Management API (`@connections`) to broadcast the message to all connected clients.
6. **Disconnection Cleanup**: Upon closing the browser tab or dropping network connection, the `$disconnect` route automatically triggers `onDisconnect` to purge the dead `connectionId` from DynamoDB.

---

#### Key Architectural Benefits
- **Zero Server Idle Cost**: No dedicated EC2 or VPS instances running 24/7.
- **Automatic Scalability**: AWS Lambda and DynamoDB scale seamlessly with actual message volume.
- **Cost Optimization**: Operates well within the **AWS Free Tier**.