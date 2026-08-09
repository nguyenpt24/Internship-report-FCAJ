---
title: "End-to-End Testing & Cleanup"
date: 2026-08-04
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

### 5.8. End-to-End Real-Time Browser Testing & Resource Cleanup Scripts

#### Module Goal
Test real-time message delivery across 2 independent browser tabs and execute AWS CLI cleanup commands to purge all lab resources upon completion.

---

#### 1. End-to-End Real-Time Chat Testing
1. Open the CloudFront Domain URL (or open `index.html`) on **Browser Tab 1 (User A)**.
2. Open the URL in an **Incognito Window or Browser Tab 2 (User B)**.
3. Check status: Both tabs display `Connected Online (WebSocket Active)`.
4. Type a message on Tab 1 and click **Send**.
5. **Verification**: The message arrives almost instantaneously (< 100ms) on both Tab 1 and Tab 2.

![Real-time chat verification screenshot across 2 browser tabs](/images/5.8-testing-chat-two-tabs.png)

---

#### 2. Resource Cleanup Scripts
To avoid unexpected charges post-evaluation, execute the following AWS CLI commands in your terminal to delete all created resources:

```bash
# 1. Delete WebSocket API Gateway
aws apigatewayv2 delete-api --api-id <YOUR-API-ID>

# 2. Delete AWS Lambda functions
aws lambda delete-function --function-name onConnect
aws lambda delete-function --function-name onDisconnect
aws lambda delete-function --function-name sendMessage

# 3. Delete Amazon DynamoDB table
aws dynamodb delete-table --table-name WebSocketConnections

# 4. Force delete S3 Frontend Bucket
aws s3 rb s3://my-serverless-chat-frontend-2026 --force

# 5. Detach Policies and Delete IAM Role ChatAppLambdaRole
aws iam detach-role-policy --role-name ChatAppLambdaRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam detach-role-policy --role-name ChatAppLambdaRole --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess
aws iam detach-role-policy --role-name ChatAppLambdaRole --policy-arn arn:aws:iam::aws:policy/AmazonAPIGatewayInvokeFullAccess
aws iam delete-role-policy --role-name ChatAppLambdaRole --policy-name WebSocketManageConnectionsPolicy
aws iam delete-role --role-name ChatAppLambdaRole
```

![Terminal screenshot confirming deletion of API Gateway, Lambda, and DynamoDB resources via AWS CLI](/images/5.8-cleanup-aws-cli-result(1).png)

![Terminal screenshot confirming deletion of S3 Bucket and IAM Role via AWS CLI](/images/5.8-cleanup-aws-cli-result(2).png)
