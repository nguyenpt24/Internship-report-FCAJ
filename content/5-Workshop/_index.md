---
title: "Workshop"
date: 2026-08-04
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Workshop: Building a Serverless Real-Time Chat Application on AWS
## (Serverless Real-Time Chat Application on AWS)

---

### Workshop Overview

This hands-on Workshop provides a step-by-step tutorial on building and deploying a complete **Serverless Real-Time Chat Application** on AWS Cloud.

The solution integrates bi-directional **WebSocket protocol (Amazon API Gateway)** with event-driven compute (**AWS Lambda**), high-speed NoSQL database storage (**Amazon DynamoDB**), static frontend hosting (**Amazon S3 + CloudFront CDN**), user authentication (**Amazon Cognito**), and centralized monitoring (**Amazon CloudWatch**).

---

### Workshop Modules Checklist:

1. **[5.1. Architecture Overview](5.1-workshop-overview/)**: WebSocket Serverless concept & Data flow diagram.
2. **[5.2. Prerequisites & IAM Roles](5.2-prerequiste/)**: Configure AWS CLI & create IAM Role for Lambda.
3. **[5.3. Create Amazon DynamoDB Table](5.3-dynamodb/)**: Provision `WebSocketConnections` table for active connections.
4. **[5.4. Author AWS Lambda Handlers](5.4-lambda-backend/)**: Write Node.js code for `onConnect`, `onDisconnect`, and `sendMessage`.
5. **[5.5. Configure WebSocket API Gateway](5.5-api-gateway/)**: Define routes `$connect`, `$disconnect`, `sendmessage` & deploy Stage `production`.
6. **[5.6. Deploy Web Frontend & CloudFront](5.6-frontend-s3/)**: Host Chat Frontend (`index.html`) on S3 and distribute via CloudFront.
7. **[5.7. Integrate Cognito & CloudWatch Alarms](5.7-cognito-cloudwatch/)**: User authentication & automated CloudWatch error alarms.
8. **[5.8. End-to-End Testing & Cleanup](5.8-testing-cleanup/)**: Real-time chat testing across browser tabs & resource cleanup scripts.