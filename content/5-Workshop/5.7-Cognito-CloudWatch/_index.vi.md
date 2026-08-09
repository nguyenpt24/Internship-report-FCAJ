---
title: "Tích hợp Cognito & CloudWatch"
date: 2026-08-04
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

### 5.7. Tích Hợp Cognito Authentication & Cấu Hình Giám Sát CloudWatch

#### Mục tiêu bài lab
Tạo **Amazon Cognito User Pool** để xác thực người dùng đăng nhập ứng dụng Chat, kiểm tra log thực thi trong **CloudWatch Logs** và cấu hình **CloudWatch Alarm** cảnh báo khi có lỗi phát sinh trong hàm Lambda.

---

#### 1. Tạo Amazon Cognito User Pool & App Client
1. Mở dịch vụ **Amazon Cognito Console**.
2. Ở menu bên trái chọn **User pools** (chú ý: chọn *User pools*, KHÔNG chọn *Identity pools*) ➔ Nhấn nút **Create user pool**.
3. Cấu hình các thông số:
   - **Application type**: Chọn **Single-page application (SPA)**.
   - **Name your application (App client name)**: Nhập `ChatAppWebClient`.
   - **Options for sign-in identifiers**: Tích chọn **Email** (nếu tích chọn Username, tại ô *Required attributes for sign-up* bên dưới bạn nhấp chọn **email** để xóa báo lỗi đỏ).
   - **Return URL / Các mục khác**: Có thể để trống.
4. Nhấn nút màu cam **Create user directory** (hoặc **Create user pool**) ở góc dưới cùng bên phải để hoàn tất.

![Màn hình khởi tạo Amazon Cognito User Pool và App Client](/images/5.7-cognito-user-pool-created.png)

---

##### 2. Cấu hình Tích hợp với WebSocket API Gateway (Tùy chọn nâng cao)
   - Khi kích hoạt tính năng bảo mật, Client sẽ gửi Cognito ID Token qua tham số URL khi mở kết nối WebSocket:
     `wss://<YOUR-API-ID>.execute-api.us-east-1.amazonaws.com/production?Authorization=<COGNITO_ID_TOKEN>`
   - Trên API Gateway WebSocket Console, truy cập mục **Authorizers** ➔ Thêm **Lambda Request Authorizer** để giải mã và kiểm tra tính hợp lệ của Cognito JWT Token trước khi chuyển tiếp yêu cầu kết nối tới route `$connect`.

---

#### 2. Kiểm tra CloudWatch Logs & Thiết lập CloudWatch Alarm
1. **Kiểm tra CloudWatch Logs**:
   - Tại menu bên trái, mở mục **Logs** ➔ Chọn **Log Management** (hoặc Log groups).
   - Chọn Log Group `/aws/lambda/onConnect` (hoặc `/aws/lambda/sendMessage`) để xem chi tiết log thực thi và thời gian phản hồi của hàm Lambda.

![Chi tiết dữ liệu Log Stream trong CloudWatch Logs Group /aws/lambda/onConnect hoặc /aws/lambda/sendMessage](/images/5.7-cloudwatch-log-stream.png)

2. **Thiết lập CloudWatch Alarm báo lỗi (Wizard 4 bước)**:
   - **Step 1 (Specify metric and conditions)**:
     - Giữ mặc định `Metrics` & `Classic` ➔ Nhấn nút màu cam **Select metric**.
     - Chọn **AWS/Lambda** ➔ **By Function Name** ➔ Tích chọn metric **Errors** của hàm `sendMessage` (hoặc `onConnect`) ➔ Nhấn **Select metric**.
     - Mục Conditions: Chọn **Greater/Equal (`>=`)** và nhập `1` ➔ Nhấn **Next**.
   - **Step 2 (Configure actions)**: Nhấn **Remove** tại mục Notification (để không cần cấu hình email SNS) ➔ Nhấn **Next**.
   - **Step 3 (Add alarm details)**: Nhập tên Alarm: `HighLambdaErrorAlert` ➔ Nhấn **Next**.
   - **Step 4 (Preview and create)**: Kiểm tra thông tin và nhấn **Create alarm** (lưu ý: ngay sau khi tạo, Alarm sẽ hiển thị trạng thái *Insufficient data* trong 2–5 phút đầu tiên trước khi cập nhật dữ liệu metric sang trạng thái *OK*).

![Màn hình cấu hình CloudWatch Alarm HighLambdaErrorAlert ở trạng thái OK hoặc Insufficient data](/images/5.7-cloudwatch-alarm-ok.png)
