---
title: "Prerequisites & IAM Roles"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

### 5.2. Prerequisites & Creating IAM Role for Lambda

#### Module Goal
Create the IAM Role `ChatAppLambdaRole` granting AWS Lambda permission to write CloudWatch logs, access DynamoDB tables, and broadcast messages via API Gateway WebSocket connections.

---

#### Step-by-Step Instructions

##### Step 1: Verify AWS CLI Configuration
Open your terminal and verify your active AWS CLI credentials:

```bash
aws sts get-caller-identity
```

![Output of aws sts get-caller-identity verifying AWS identity](/images/5.2-aws-cli-identity.png)

---

##### Step 2: Create IAM Role via AWS Management Console
1. Open the **IAM Management Console**.
2. Select **Roles** ➔ Click **Create role**.
3. Choose **AWS service**, select **Lambda** under Use case ➔ Click **Next**.

![Choosing Lambda trusted entity type for IAM Role](/images/5.2-iam-role-trusted-entity.png)

4. In the **Add permissions** step, search and attach the following policies:
   - `AWSLambdaBasicExecutionRole` (CloudWatch logging permissions)
   - `AmazonDynamoDBFullAccess` (DynamoDB table read/write permissions)
   - `AmazonAPIGatewayInvokeFullAccess` (API Gateway invoke permissions)

![Attaching Permissions Policies to IAM Role](/images/5.2-iam-role-policies.png)

5. Enter Role Name: `ChatAppLambdaRole` ➔ Click **Create role**.

6. **Add Inline Policy for WebSocket Connection Management (`execute-api:ManageConnections`)**:
   - Open the newly created `ChatAppLambdaRole`.
   - Under the **Permissions** tab, click **Add permissions** ➔ **Create inline policy**.
   - Select the **JSON** tab and paste the following policy statement:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Action": [
                     "execute-api:ManageConnections"
                 ],
                 "Resource": "arn:aws:execute-api:*:*:*/*/*/@connections/*"
             }
         ]
     }
     ```
   - Name the Policy `WebSocketManageConnectionsPolicy` ➔ Click **Create policy**.

![Successful creation of inline policy WebSocketManageConnectionsPolicy](/images/5.2-iam-inline-policy.png)