---
title: "Deploy Web Frontend & CloudFront"
date: 2026-08-04
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

### 5.6. Deploy Web Chat Client on Amazon S3 & CloudFront CDN

#### Module Goal

Build the Web Client interface (`index.html`), upload assets to Amazon S3, and configure an Amazon CloudFront Distribution to distribute the Chat application globally over secure HTTPS.

---

#### 1. Web Frontend Client Source Code (`index.html`)

Create `index.html` locally and paste the code below (replace `<YOUR-API-ID>` with your live WebSocket WSS URL):

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Serverless Real-Time Chat App</title>
    <style>
      body {
        font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
        background: #0f172a;
        color: #f8fafc;
        padding: 20px;
      }
      .chat-container {
        max-width: 600px;
        margin: 0 auto;
        background: #1e293b;
        border-radius: 12px;
        padding: 20px;
        box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
      }
      h2 {
        text-align: center;
        color: #38bdf8;
        margin-bottom: 20px;
      }
      #status {
        text-align: center;
        font-size: 14px;
        font-weight: bold;
        margin-bottom: 15px;
      }
      .online {
        color: #4ade80;
      }
      .offline {
        color: #f87171;
      }
      #messages {
        height: 350px;
        overflow-y: auto;
        background: #0f172a;
        border-radius: 8px;
        padding: 15px;
        margin-bottom: 15px;
        display: flex;
        flex-direction: column;
        gap: 8px;
      }
      .msg {
        padding: 8px 12px;
        border-radius: 6px;
        font-size: 14px;
        max-width: 80%;
        word-break: break-word;
      }
      .received {
        background: #334155;
        color: #f8fafc;
        align-self: flex-start;
      }
      .sent {
        background: #0284c7;
        color: #ffffff;
        align-self: flex-end;
        text-align: right;
      }
      .username-group {
        display: flex;
        gap: 10px;
        margin-bottom: 10px;
        align-items: center;
      }
      .username-group label {
        font-size: 14px;
        font-weight: bold;
        color: #94a3b8;
      }
      input[type="text"] {
        padding: 12px;
        border-radius: 6px;
        border: 1px solid #475569;
        background: #0f172a;
        color: #fff;
      }
      #usernameInput {
        width: 140px;
      }
      .input-group {
        display: flex;
        gap: 10px;
      }
      #messageInput {
        flex: 1;
      }
      button {
        padding: 12px 24px;
        background: #0284c7;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-weight: bold;
      }
      button:hover {
        background: #0369a1;
      }
    </style>
  </head>
  <body>
    <div class="chat-container">
      <h2>AWS Serverless Real-Time Chat</h2>
      <div id="status" class="offline">Connecting WebSocket...</div>
      <div class="username-group">
        <label for="usernameInput">Display Name:</label>
        <input
          type="text"
          id="usernameInput"
          placeholder="Enter name..."
          value="User A"
        />
      </div>
      <div id="messages"></div>
      <div class="input-group">
        <input
          type="text"
          id="messageInput"
          placeholder="Type a message..."
          onkeydown="if(event.key==='Enter') sendMessage()"
        />
        <button onclick="sendMessage()">Send</button>
      </div>
    </div>

    <script>
      // Replace with your live WebSocket URL
      const WSS_URL =
        "wss://<YOUR-API-ID>.execute-api.us-east-1.amazonaws.com/production";
      let socket;

      function connect() {
        socket = new WebSocket(WSS_URL);

        socket.onopen = () => {
          document.getElementById("status").innerText =
            "Connected Online (WebSocket Active)";
          document.getElementById("status").className = "online";
        };

        socket.onmessage = (event) => {
          const data = JSON.parse(event.data);
          const msgBox = document.getElementById("messages");
          const div = document.createElement("div");
          div.className = "msg received";
          div.innerText = data.message;
          msgBox.appendChild(div);
          msgBox.scrollTop = msgBox.scrollHeight;
        };

        socket.onclose = () => {
          document.getElementById("status").innerText =
            "Disconnected Offline. Retrying...";
          document.getElementById("status").className = "offline";
          setTimeout(connect, 3000);
        };
      }

      function sendMessage() {
        const input = document.getElementById("messageInput");
        const usernameInput = document.getElementById("usernameInput");
        const username = usernameInput.value.trim() || "Anonymous";
        if (input.value.trim() !== "" && socket.readyState === WebSocket.OPEN) {
          const fullText = username + ": " + input.value.trim();
          const payload = JSON.stringify({
            action: "sendmessage",
            data: fullText,
          });
          socket.send(payload);
          input.value = "";
        }
      }

      connect();
    </script>
  </body>
</html>
```

---

#### 2. Amazon S3 Hosting & CloudFront Distribution with Origin Access Control (OAC)

##### Step 1: Create S3 Bucket and Upload `index.html`

1. Open **Amazon S3 Console** ➔ Click **Create bucket**.
2. Enter Bucket name: `my-serverless-chat-frontend-2026` ➔ Keep default settings (Block all public access **Enabled**) ➔ Click **Create bucket**.
3. Select the created Bucket and upload `index.html` to the root of the Bucket.

![index.html uploaded to S3 Bucket](/images/5.6-s3-upload-index-html.png)

##### Step 2: Create CloudFront Distribution & Origin Access Control (OAC) (5-Step Wizard Interface)

1. Open **Amazon CloudFront Console** ➔ Click **Create distribution**.
2. **Step 1: Get started**:
   - **Distribution name**: Enter `my-serverless-chat-frontend-distribution`.
   - **Distribution type**: Select **Single website or app** (Default).
   - Click **Next**.
3. **Step 2: Specify origin**:
   - **Origin domain**: Select your S3 Bucket (`my-serverless-chat-frontend-2026.s3.amazonaws.com`).
   - **Origin access**: Select **Origin access control settings (recommended)** ➔ Click **Create control setting** ➔ Click **Create** to generate OAC.
   - Click **Next**.
4. **Step 3 (Enable security)**:
   - Select **Do not enable security protections** (to bypass AWS WAF creation and avoid the $14/month charges for lab testing).
   - Click **Next**.
5. **Step 4 (Review and create)**:
   - Review general settings (Billing displays `$0/month`, Security displays `None`).
   - Click the orange **Create distribution** button at the bottom right to finalize creation.

![Creating CloudFront Distribution with S3 Origin and OAC](/images/5.6-cloudfront-create-distribution.png)

##### Step 3: Configure Default Root Object (`index.html`) & Update S3 Bucket Policy

1. **Configure Default Root Object**:
   - On the Distribution details page (**General** tab), under **Settings** panel, click **Edit**.
   - Under **Default root object**, type `index.html` ➔ Click **Save changes**.
2. **Verify / Update S3 Bucket Policy**:
   - Switch to the **Origins** tab ➔ Select the S3 Origin row ➔ Click **Edit**.
   - Click **Copy policy** (or review the auto-generated bucket policy statement).
   - Open the **Amazon S3 Console** ➔ Select your Bucket `my-serverless-chat-frontend-2026` ➔ Switch to the **Permissions** tab.
   - Under **Bucket policy**, click **Edit** and paste the JSON policy ➔ Click **Save changes**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": {
       "Sid": "AllowCloudFrontServicePrincipalReadOnly",
       "Effect": "Allow",
       "Principal": {
         "Service": "cloudfront.amazonaws.com"
       },
       "Action": "s3:GetObject",
       "Resource": "arn:aws:s3:::my-serverless-chat-frontend-2026/*",
       "Condition": {
         "StringEquals": {
           "AWS:SourceArn": "arn:aws:cloudfront::<ACCOUNT-ID>:distribution/<DISTRIBUTION-ID>"
         }
       }
     }
   }
   ```

![S3 Bucket Policy successfully updated for CloudFront OAC](/images/5.6-s3-bucket-policy-oac.png)

##### Step 4: Retrieve CloudFront Domain Name & Test Application

1. On the Distribution details page (**General** tab), copy your **Distribution domain name**.
2. Open a web browser and navigate to `https://<distribution-name>.cloudfront.net` to verify your real-time chat application over HTTPS.

![CloudFront Distribution Domain Name URL display and successful load test](/images/5.6-cloudfront-domain-test.png)
