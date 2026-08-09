---
title: "Project Proposal"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Project Proposal: Building a Serverless Real-Time Chat Application on AWS

---

### 1. Proposal Overview
This project aims to build a low-latency real-time messaging application capable of scaling automatically with user traffic while optimizing operational costs using AWS Cloud services. Combining the WebSocket protocol with a Serverless architecture eliminates the need for 24/7 server maintenance, satisfying technical requirements for high availability, security, and centralized monitoring.

---

### 2. Problem Statement and Objectives

#### Current Challenges
- Traditional chat applications rely on socket servers (such as Node.js/Socket.io) running 24/7 on EC2 or VPS instances, incurring fixed running costs even during zero-traffic periods.
- Configuring Auto Scaling for bi-directional socket connections during traffic spikes can be complex.
- Requires ongoing administrative effort for operating system updates and server maintenance.

#### Solution Approach
- **Serverless Architecture**: Utilizes **Amazon API Gateway (WebSocket API)** to manage persistent bi-directional connections, **AWS Lambda** to handle event logic, and **Amazon DynamoDB** to store active connections and message history.
- **Hosting and Storage**: The web user interface (HTML/CSS/JS) is stored in **Amazon S3** and distributed via **Amazon CloudFront**. User authentication is integrated using **Amazon Cognito**.
- **System Monitoring**: Execution logs and performance metrics are tracked via **Amazon CloudWatch**.

#### Key Benefits
- **Cost Optimization**: Pay-per-use pricing makes maximum use of the AWS Free Tier, minimizing operating costs during development.
- **Low Latency**: Persistent WebSocket connections allow messages to be sent and received almost instantly.
- **Automatic Scaling**: The system scales seamlessly as concurrent connections increase without manual server provisioning.

---

### 3. Architecture Diagram

![AWS Serverless Real-Time Chat App Solution Architecture Diagram](/images/serverless-chat-architecture.png)

```
 [ Web Client Browser ]  <--- HTTPS ---> [ Amazon CloudFront / S3 ] (Frontend Hosting)
           |
       WebSocket (WSS)
           |
           v
    [ Amazon API Gateway ] (WebSocket API)
     /         |         \
 $connect  $disconnect  sendmessage
    |          |            |
    v          v            v
      [ AWS Lambda Handlers ]
               |
               v
      [ Amazon DynamoDB ] (Tables: WebSocketConnections, ChatMessages)
               |
               v
      [ Amazon CloudWatch ] (Logs, Metrics, Alarms)
```

#### AWS Services Utilized:
1. **Amazon API Gateway (WebSocket API)**: Manages persistent bi-directional WebSocket connections.
2. **AWS Lambda**: Executes handler code for `$connect`, `$disconnect`, and `sendmessage` routes.
3. **Amazon DynamoDB**: Stores connection data (`connectionId`) and message contents.
4. **Amazon S3 & CloudFront**: Stores and accelerates delivery of the web client interface.
5. **Amazon Cognito**: Handles user registration, authentication, and token verification.
6. **Amazon CloudWatch**: Records system logs and triggers alarms on errors.

---

### 4. Implementation Plan

| Phase | Tasks | Deliverables |
| --- | --- | --- |
| **Phase 1** | Requirement analysis, architecture design, and DynamoDB table schema modeling | Architecture Diagram & DynamoDB Schema |
| **Phase 2** | Initialize DynamoDB table and author Lambda functions (`onConnect`, `onDisconnect`, `sendMessage`) | Lambda Event Handler Functions |
| **Phase 3** | Configure WebSocket API on API Gateway, set up routes, and create Deployment Stage | WebSocket Endpoint (`wss://...`) |
| **Phase 4** | Build Web Client interface (`index.html`), connect WebSocket, and deploy to S3/CloudFront | Live Web Chat on CloudFront |
| **Phase 5** | Integrate Amazon Cognito authentication, configure CloudWatch Logs & Alarms | Authenticated & Monitored System |
| **Phase 6** | End-to-End messaging test, CloudWatch log verification, architecture review, and cleanup script preparation | Completed Report & Cleanup Script |

---

### 5. Budget Estimation

The system is designed to operate within the **AWS Free Tier** limits:

- **AWS Lambda**: 1,000,000 free requests/month and 3.2 million seconds of compute time.
- **Amazon DynamoDB**: 25 GB of free NoSQL storage.
- **Amazon API Gateway**: 1,000,000 free WebSocket connection minutes and 1,000,000 messages/month.
- **Amazon S3 & CloudFront**: 5 GB of free S3 storage and 1 TB of CloudFront data transfer.
- **Amazon CloudWatch**: 5 GB of free log data and 10 custom metrics.

Expected monthly cost: **$0.00 USD/month** (or under $1.00 USD if testing slightly exceeds free limits).

---

### 6. Risk Management

- **Handling Stale Connections**: When clients disconnect abruptly without deleting `connectionId` from DynamoDB.
  - *Resolution*: Configure `sendMessage` to catch `410 Gone` HTTP status codes from API Gateway and automatically remove invalid `connectionId` records from DynamoDB.
- **Access Control Security**:
  - *Resolution*: Enforce IAM Least Privilege permissions on Lambda roles and require Cognito token verification on WebSocket connections.
- **Cost Overflow Control**:
  - *Resolution*: Set an **AWS Budgets** alert at $1.00 and execute resource cleanup scripts after testing.

---

### 7. Expected Outcomes
1. Successfully build and deploy the **Serverless Real-Time Chat App** running on AWS.
2. Complete all technical requirements and milestones for the **First Cloud AI Journey** internship report.
3. Master Serverless application design and implementation principles on AWS.