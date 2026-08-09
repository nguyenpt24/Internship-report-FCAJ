---
title: "Cấu hình WebSocket API Gateway"
date: 2026-08-04
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

### 5.5. Khởi tạo và Cấu hình Amazon API Gateway (WebSocket API)

#### Mục tiêu bài lab
Tạo **WebSocket API** trên Amazon API Gateway, định tuyến các route sự kiện (`$connect`, `$disconnect`, `sendmessage`) tới các hàm Lambda tương ứng và triển khai Stage `production`.

---

#### Hướng dẫn thực hiện

##### Bước 1: Tạo WebSocket API
1. Mở dịch vụ **Amazon API Gateway Console**.
2. Chọn **Build** tại mục **WebSocket API**.
3. Điền thông tin API:
   - **API name**: `RealtimeChatWebSocketAPI`
   - **Route selection expression**: `$request.body.action`
   - **IP address type**: Giữ mặc định **IPv4** (không cần thay đổi).

![Màn hình tạo WebSocket API với Route selection expression](/images/5.5-apigateway-websocket-create.png)

---

##### Bước 2: Thêm các Route sự kiện
1. Tại phần Predefined routes: Tích chọn **$connect** và **$disconnect**.
2. Tại phần Custom routes: Thêm route key `sendmessage`.

![Màn hình cấu hình các tuyến đường $connect, $disconnect và sendmessage](/images/5.5-apigateway-routes-config.png)

---

##### Bước 3: Đóng gói tích hợp với AWS Lambda (Integrations)
1. Ghép nối Route **$connect** ➔ Hàm Lambda `onConnect`.
2. Ghép nối Route **$disconnect** ➔ Hàm Lambda `onDisconnect`.
3. Ghép nối Route **sendmessage** ➔ Hàm Lambda `sendMessage`.

![Màn hình ghép nối các Route với hàm Lambda tương ứng](/images/5.5-apigateway-lambda-integration.png)

---

##### Bước 4: Triển khai Stage và Lấy đường dẫn WSS URL
1. Đặt tên Stage: `production` ➔ Nhấn **Next** và chọn **Create and Deploy**.
2. Lưu lại đường dẫn **WebSocket URL** (có dạng `wss://<api-id>.execute-api.us-east-1.amazonaws.com/production`).

![Đường dẫn WebSocket Endpoint WSS hiển thị trên API Gateway Console](/images/5.5-apigateway-wss-endpoint.png)
