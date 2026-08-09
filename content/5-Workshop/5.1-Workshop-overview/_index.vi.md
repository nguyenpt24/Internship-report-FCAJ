---
title: "Tổng quan kiến trúc"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### 5.1. Tổng quan dự án & Sơ đồ kiến trúc WebSocket Serverless

#### Giới thiệu bài lab
Trong bài thực hành này, chúng ta sẽ tìm hiểu về mô hình kết nối hai chiều (bi-directional communication) thông qua giao thức **WebSocket** và lý do tại sao kiến trúc **Serverless trên AWS** lại là giải pháp tối ưu nhất cho ứng dụng Chat thời gian thực.

---

#### Mô hình Luồng xử lý dữ liệu (Data Flow)

![Sơ đồ kiến trúc tổng quan hệ thống Chat Real-Time trên AWS](/images/serverless-chat-architecture.png)

1. **Client Kết nối**: Người dùng mở trang web Chat Frontend. Trình duyệt gửi yêu cầu kết nối WebSocket qua đường dẫn WSS (`wss://...execute-api.us-east-1.amazonaws.com/production`).
2. **API Gateway tiếp nhận**: Amazon API Gateway tiếp nhận kết nối WebSocket và kích hoạt hàm Lambda `onConnect`.
3. **Lưu ID Kết nối**: Hàm `onConnect` lấy `connectionId` do API Gateway cấp và lưu vào bảng Amazon DynamoDB `WebSocketConnections`.
4. **Gửi tin nhắn**: Khi người dùng nhập nội dung và bấm Gửi, client gửi frame JSON dạng `{"action": "sendmessage", "data": "Xin chào!"}`.
5. **Phân phối tin nhắn**: API Gateway định tuyến đến hàm Lambda `sendMessage`. Hàm này đọc tất cả `connectionId` trong DynamoDB và gọi API Gateway Management API (`@connections`) để phát tin nhắn tới toàn bộ các client đang kết nối.
6. **Xử lý ngắt kết nối**: Khi ngắt kết nối hoặc đóng tab, tuyến đường `$disconnect` tự động kích hoạt hàm `onDisconnect` để xóa `connectionId` tương ứng khỏi DynamoDB.

---

#### Ưu điểm của kiến trúc này
- **Không tốn chi phí duy trì máy chủ**: Không cần thuê máy chủ EC2 hay VPS 24/7.
- **Tự động mở rộng**: AWS Lambda và DynamoDB tự động co giãn theo số lượng tin nhắn thực tế.
- **Tối ưu ngân sách**: Hoàn toàn nằm trong gói **AWS Free Tier**.
