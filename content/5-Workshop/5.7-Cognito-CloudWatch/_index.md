---
title: "Integrate Cognito & CloudWatch"
date: 2026-08-04
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

### 5.7. Integrate Cognito Authentication & Configure CloudWatch Alarms

#### Module Goal
Create an **Amazon Cognito User Pool** to manage user sign-in authentication, inspect execution logs in **CloudWatch Logs**, and configure automated **CloudWatch Alarms** for Lambda errors.

---

#### 1. Create Amazon Cognito User Pool & App Client
1. Open the **Amazon Cognito Console**.
2. On the left navigation menu, select **User pools** (make sure to select *User pools*, NOT *Identity pools*) ➔ Click **Create user pool**.
3. Configure application parameters:
   - **Application type**: Select **Single-page application (SPA)**.
   - **Name your application (App client name)**: Enter `ChatAppWebClient`.
   - **Options for sign-in identifiers**: Check **Email** (if Username is checked, select **email** in the *Required attributes for sign-up* dropdown to clear the red error).
   - **Return URL / Other options**: Can be left blank.
4. Click the orange **Create user directory** (or **Create user pool**) button at the bottom right to finalize creation.

![Creating Amazon Cognito User Pool and App Client screen](/images/5.7-cognito-user-pool-created.png)

---

##### 2. WebSocket API Gateway Integration (Advanced Option)
   - When enabling end-to-end security, the client app passes the Cognito ID Token as a query parameter during the WebSocket handshake:
     `wss://<YOUR-API-ID>.execute-api.us-east-1.amazonaws.com/production?Authorization=<COGNITO_ID_TOKEN>`
   - On the API Gateway WebSocket Console, navigate to **Authorizers** ➔ Create a **Lambda Request Authorizer** to validate the Cognito JWT Token before granting access to the `$connect` route.

---

#### 2. Inspect CloudWatch Logs & Configure CloudWatch Alarm
1. **Inspect CloudWatch Logs**:
   - On the left menu, expand **Logs** ➔ Select **Log Management** (or Log groups).
   - Select Log Group `/aws/lambda/onConnect` (or `/aws/lambda/sendMessage`) to inspect invocation logs and Lambda response latencies.

![Execution Log Stream inside CloudWatch Logs Group /aws/lambda/onConnect or /aws/lambda/sendMessage](/images/5.7-cloudwatch-log-stream.png)

2. **Configure CloudWatch Alarm for Errors (4-Step Wizard)**:
   - **Step 1 (Specify metric and conditions)**:
     - Keep defaults `Metrics` & `Classic` ➔ Click the orange **Select metric** button.
     - Select **AWS/Lambda** ➔ **By Function Name** ➔ Check the **Errors** metric for `sendMessage` (or `onConnect`) ➔ Click **Select metric**.
     - Under Conditions: Select **Greater/Equal (`>=`)** and enter `1` ➔ Click **Next**.
   - **Step 2 (Configure actions)**: Click **Remove** under Notification section ➔ Click **Next**.
   - **Step 3 (Add alarm details)**: Enter Alarm name `HighLambdaErrorAlert` ➔ Click **Next**.
   - **Step 4 (Preview and create)**: Review settings and click **Create alarm** (note: immediately after creation, the Alarm displays *Insufficient data* status for the first 2–5 minutes before updating metric evaluation to *OK* status).

![CloudWatch Alarm HighLambdaErrorAlert in OK or Insufficient data state](/images/5.7-cloudwatch-alarm-ok.png)
