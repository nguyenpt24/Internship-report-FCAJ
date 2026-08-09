---
title: "Triển khai Web Frontend & CloudFront"
date: 2026-08-04
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

### 5.6. Triển Khai Web Chat Client Trên Amazon S3 & CloudFront CDN

#### Mục tiêu bài lab

Tạo trang web Frontend (`index.html`), tải lên Amazon S3 và cấu hình Amazon CloudFront Distribution để phân phối giao diện Chat toàn cầu qua kết nối bảo mật HTTPS.

---

#### 1. Mã nguồn Giao diện Web Frontend Client (`index.html`)

Tạo tệp `index.html` trên máy tính local và dán nội dung sau (nhớ thay thế đường dẫn WebSocket WSS URL của bạn):

```html
<!DOCTYPE html>
<html lang="vi">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>AWS Serverless Real-Time Chat</title>
    <style>
      * {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
      }
      body {
        font-family:
          -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica,
          Arial, sans-serif;
        background-color: #18191a;
        color: #e4e6eb;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
        padding: 20px;
      }
      .chat-container {
        width: 100%;
        max-width: 520px;
        height: 680px;
        background-color: #242526;
        border-radius: 16px;
        box-shadow: 0 12px 32px rgba(0, 0, 0, 0.45);
        display: flex;
        flex-direction: column;
        overflow: hidden;
        border: 1px solid #3e4042;
      }
      .chat-header {
        padding: 16px 20px;
        background-color: #242526;
        border-bottom: 1px solid #3e4042;
        display: flex;
        flex-direction: column;
        gap: 12px;
      }
      .header-top {
        display: flex;
        align-items: center;
        justify-content: space-between;
      }
      .app-title {
        display: flex;
        align-items: center;
        gap: 10px;
      }
      .app-title h2 {
        font-size: 18px;
        font-weight: 700;
        color: #f5f6f7;
      }
      .messenger-icon {
        width: 28px;
        height: 28px;
        background: linear-gradient(135deg, #0084ff 0%, #00c6ff 100%);
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        color: #fff;
        font-size: 14px;
        font-weight: bold;
      }
      #status {
        font-size: 12px;
        font-weight: 600;
        padding: 4px 10px;
        border-radius: 12px;
        display: flex;
        align-items: center;
        gap: 6px;
      }
      .status-dot {
        width: 8px;
        height: 8px;
        border-radius: 50%;
        display: inline-block;
      }
      .online {
        background-color: rgba(46, 204, 113, 0.15);
        color: #2ecc71;
      }
      .online .status-dot {
        background-color: #2ecc71;
        box-shadow: 0 0 8px #2ecc71;
      }
      .offline {
        background-color: rgba(231, 76, 60, 0.15);
        color: #e74c3c;
      }
      .offline .status-dot {
        background-color: #e74c3c;
      }
      .user-bar {
        display: flex;
        align-items: center;
        gap: 10px;
        background-color: #18191a;
        padding: 8px 14px;
        border-radius: 20px;
        border: 1px solid #3e4042;
      }
      .user-bar label {
        font-size: 13px;
        color: #b0b3b8;
        font-weight: 600;
        white-space: nowrap;
      }
      .user-bar input {
        background: transparent;
        border: none;
        color: #0084ff;
        font-weight: 700;
        font-size: 14px;
        outline: none;
        width: 100%;
      }
      #messages {
        flex: 1;
        padding: 20px;
        overflow-y: auto;
        background-color: #18191a;
        display: flex;
        flex-direction: column;
        gap: 12px;
      }
      .msg-wrapper {
        display: flex;
        flex-direction: column;
        max-width: 75%;
      }
      .msg-wrapper.sent {
        align-self: flex-end;
        align-items: flex-end;
      }
      .msg-wrapper.received {
        align-self: flex-start;
        align-items: flex-start;
      }
      .sender-name {
        font-size: 11px;
        color: #b0b3b8;
        margin-bottom: 4px;
        margin-left: 10px;
        font-weight: 600;
      }
      .msg-bubble {
        padding: 10px 16px;
        font-size: 14px;
        line-height: 1.45;
        word-break: break-word;
      }
      .sent .msg-bubble {
        background: linear-gradient(135deg, #0084ff 0%, #00c6ff 100%);
        color: #ffffff;
        border-radius: 18px 18px 4px 18px;
      }
      .received .msg-bubble {
        background-color: #3e4042;
        color: #e4e6eb;
        border-radius: 18px 18px 18px 4px;
      }
      .chat-footer {
        padding: 14px 16px;
        background-color: #242526;
        border-top: 1px solid #3e4042;
        display: flex;
        gap: 10px;
        align-items: center;
      }
      .input-wrapper {
        flex: 1;
        background-color: #3a3b3c;
        border-radius: 20px;
        padding: 8px 16px;
        display: flex;
        align-items: center;
      }
      .input-wrapper input {
        width: 100%;
        background: transparent;
        border: none;
        color: #e4e6eb;
        font-size: 14px;
        outline: none;
      }
      .send-btn {
        width: 40px;
        height: 40px;
        border-radius: 50%;
        background: linear-gradient(135deg, #0084ff 0%, #00c6ff 100%);
        border: none;
        color: #ffffff;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
      }
      .send-btn svg {
        width: 18px;
        height: 18px;
        fill: currentColor;
        margin-left: 2px;
      }
    </style>
  </head>
  <body>
    <div class="chat-container">
      <div class="chat-header">
        <div class="header-top">
          <div class="app-title">
            <div class="messenger-icon">⚡</div>
            <h2>AWS Serverless Chat</h2>
          </div>
          <div id="status" class="offline">
            <span class="status-dot"></span>
            <span id="status-text">Đang kết nối...</span>
          </div>
        </div>
        <div class="user-bar">
          <label for="usernameInput">Tên người dùng:</label>
          <input
            type="text"
            id="usernameInput"
            placeholder="Nhập tên..."
            value="User A"
          />
        </div>
      </div>
      <div id="messages"></div>
      <div class="chat-footer">
        <div class="input-wrapper">
          <input
            type="text"
            id="messageInput"
            placeholder="Nhập tin nhắn..."
            onkeydown="if(event.key==='Enter') sendMessage()"
          />
        </div>
        <button class="send-btn" onclick="sendMessage()">
          <svg viewBox="0 0 24 24">
            <path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z" />
          </svg>
        </button>
      </div>
    </div>

    <script>
      const WSS_URL =
        "wss://<YOUR-API-ID>.execute-api.us-east-1.amazonaws.com/production";
      let socket;
      const myClientId = "usr_" + Math.random().toString(36).substring(2, 8);

      function connect() {
        socket = new WebSocket(WSS_URL);
        socket.onopen = () => {
          document.getElementById("status").className = "online";
          document.getElementById("status-text").innerText = "Online (Active)";
        };
        socket.onmessage = (event) => {
          const data = JSON.parse(event.data);
          let msgObj;
          try {
            msgObj =
              typeof data.message === "string"
                ? JSON.parse(data.message)
                : data.message;
          } catch (e) {
            const str = String(data.message || "");
            const parts = str.split(/:\s*(.+)/);
            msgObj = {
              username: parts.length > 1 ? parts[0] : "User",
              text: parts.length > 1 ? parts[1] : str,
              clientId: "unknown",
            };
          }
          const isMe =
            msgObj.clientId === myClientId ||
            msgObj.username ===
              document.getElementById("usernameInput").value.trim();
          appendMessage(msgObj, isMe);
        };
        socket.onclose = () => {
          document.getElementById("status").className = "offline";
          document.getElementById("status-text").innerText =
            "Offline (Đang thử lại...)";
          setTimeout(connect, 3000);
        };
      }

      function appendMessage(msgObj, isMe) {
        const msgBox = document.getElementById("messages");
        const wrapper = document.createElement("div");
        wrapper.className = "msg-wrapper " + (isMe ? "sent" : "received");
        if (!isMe && msgObj.username) {
          const nameLabel = document.createElement("div");
          nameLabel.className = "sender-name";
          nameLabel.innerText = msgObj.username;
          wrapper.appendChild(nameLabel);
        }
        const bubble = document.createElement("div");
        bubble.className = "msg-bubble";
        bubble.innerText = msgObj.text || msgObj.message || "";
        wrapper.appendChild(bubble);
        msgBox.appendChild(wrapper);
        msgBox.scrollTop = msgBox.scrollHeight;
      }

      function sendMessage() {
        const input = document.getElementById("messageInput");
        const usernameInput = document.getElementById("usernameInput");
        const text = input.value.trim();
        const username = usernameInput.value.trim() || "User";

        if (text !== "" && socket.readyState === WebSocket.OPEN) {
          const payloadObj = {
            clientId: myClientId,
            username: username,
            text: text,
          };
          const payload = JSON.stringify({
            action: "sendmessage",
            data: JSON.stringify(payloadObj),
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

#### 2. Hosting trên Amazon S3 & Phân phối CloudFront với Origin Access Control (OAC)

##### Bước 1: Tạo S3 Bucket và Tải lên `index.html`

1. Mở **Amazon S3 Console** ➔ Nhấn **Create bucket**.
2. Đặt tên Bucket: `my-serverless-chat-frontend-2026` ➔ Giữ nguyên mặc định (Block all public access là **Enabled**) ➔ Nhấn **Create bucket**.
3. Chọn Bucket vừa tạo, tải tệp `index.html` lên root của Bucket.

![Tệp index.html được tải lên S3 Bucket thành công](/images/5.6-s3-upload-index-html.png)

##### Bước 2: Khởi tạo CloudFront Distribution & Origin Access Control (OAC) (Giao diện Wizard 5 bước)

1. Mở **Amazon CloudFront Console** ➔ Nhấn **Create distribution**.
2. **Bước 1 (Step 1: Get started)**:
   - **Distribution name**: Nhập `my-serverless-chat-frontend-distribution`.
   - **Distribution type**: Chọn **Single website or app** (Mặc định).
   - Nhấn **Next**.
3. **Bước 2 (Step 2: Specify origin)**:
   - **Origin domain**: Nhấp chọn S3 Bucket của bạn (`my-serverless-chat-frontend-2026.s3.amazonaws.com`).
   - **Origin access**: Chọn **Origin access control settings (recommended)** ➔ Nhấn **Create control setting** ➔ Nhấn **Create** để sinh OAC.
   - Nhấn **Next**.
4. **Bước 3 (Step 3: Enable security)**:
   - Chọn tùy chọn **Do not enable security protections** (để không bật WAF, tránh phát sinh chi phí 14 USD/tháng cho môi trường lab/thử nghiệm).
   - Nhấn **Next**.
5. **Bước 4 (Step 4: Review and create)**:
   - Kiểm tra lại thông tin tổng quan (Billing hiển thị `$0/month`, Security hiển thị `None`).
   - Nhấn nút màu cam **Create distribution** ở góc dưới bên phải để hoàn tất khởi tạo.

![Màn hình tạo CloudFront Distribution trỏ đến S3 Origin với OAC](/images/5.6-cloudfront-create-distribution.png)

##### Bước 3: Cấu hình Default Root Object (`index.html`) & Cập nhật S3 Bucket Policy

1. **Cấu hình Default Root Object**:
   - Tại trang chi tiết Distribution (tab **General**), trong khung **Settings**, nhấn nút **Edit**.
   - Tại mục **Default root object**, nhập `index.html` ➔ Nhấn **Save changes**.
2. **Kiểm tra / Cập nhật S3 Bucket Policy**:
   - Chuyển sang tab **Origins** ➔ Tích chọn dòng Origin S3 ➔ Nhấn **Edit**.
   - Nhấn nút **Copy policy** (hoặc kiểm tra câu lệnh Bucket Policy được sinh tự động).
   - Mở **Amazon S3 Console** ➔ Chọn Bucket `my-serverless-chat-frontend-2026` ➔ Chuyển sang tab **Permissions**.
   - Tại mục **Bucket policy**, chọn **Edit** và dán chính sách JSON (có dạng bên dưới) ➔ Nhấn **Save changes**:
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

![S3 Bucket Policy được cập nhật thành công cho CloudFront OAC](/images/5.6-s3-bucket-policy-oac.png)

##### Bước 4: Lấy đường dẫn CloudFront Domain Name và Chạy thử

1. Tại trang chi tiết Distribution (tab **General**), sao chép đường dẫn **Distribution domain name** .
2. Mở trình duyệt web và truy cập đường dẫn của CloudFront Distribution để kiểm tra giao diện Chat Real-Time.

![Màn hình hiển thị đường dẫn CloudFront Distribution Domain Name và chạy thử thành công](/images/5.6-cloudfront-domain-test.png)
