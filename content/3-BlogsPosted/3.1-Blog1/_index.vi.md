---
title: "Blog 1: Tối Ưu Hiệu Năng AWS Lambda"
date: 2026-08-04
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Tối Ưu Hiệu Năng & Xử Lý Cold Start Trong AWS Lambda

---

### 1. Bối Cảnh: Bản Chất Của Serverless Compute & Thách Thức Cold Start

AWS Lambda cho phép chạy code mà không cần quản lý máy chủ (Event-driven Serverless Compute). Khi có yêu cầu (request), AWS tự động cấp phát tài nguyên container để thực thi mã nguồn và tự động giải phóng khi hoàn tất.

Tuy nhiên, khi một hàm Lambda không được gọi trong một khoảng thời gian (hoặc khi lưu lượng truy cập tăng đột biến đòi hỏi tạo thêm container mới), AWS phải tạo mới môi trường thực thi (Execution Environment). Việc này tạo ra một khoảng trễ khởi tạo ban đầu gọi là **Cold Start**.

![Sơ đồ so sánh luồng xử lý Cold Start và Warm Execution trong AWS Lambda](/images/blog1.png)

---

### 2. Các Chiến Lược Tối Ưu Hiệu Năng Hàng Đầu (Performance Best Practices)

Chu kỳ sống của một Lambda Function gồm 3 giai đoạn: **Init Phase** (tải code, khởi tạo runtime & code ngoài handler), **Invoke Phase** (thực thi code trong handler) và **Shutdown Phase**.

1. **Tận dụng Execution Context Reuse (Tái sử dụng kết nối)**:
   Khởi tạo các kết nối Database (DynamoDB client, SDK instances) bên ngoài hàm `lambda_handler` để dùng lại ở các lần gọi sau (Warm Execution).
2. **Tối ưu hóa dung lượng gói Deployment Package**:
   Chỉ nhúng các thư viện tối thiểu cần thiết, loại bỏ tài nguyên thừa để giảm thời gian tải container trong Init Phase.
3. **Sử dụng Provisioned Concurrency**:
   Yêu cầu AWS duy trì sẵn một số lượng container luôn ở trạng thái "ấm" (Warm Containers), loại bỏ 100% độ trễ Cold Start cho các tác vụ đòi hỏi thời gian phản hồi tức thì.

---

### 3. Mã Nguồn Mẫu Viết Theo Chuẩn Warm Context Reuse (Python Boto3)

```python
import boto3
import os
import json

# 1. KHỞI TẠO BÊN NGOÀI HANDLER (Lưu vào Memory của Execution Environment)
# Khởi tạo kết nối DynamoDB client 1 lần duy nhất trong Init Phase
dynamodb = boto3.resource('dynamodb')
table_name = os.environ['TABLE_NAME']
table = dynamodb.Table(table_name)

def lambda_handler(event, context):
    # 2. CHỈ THỰC THI LOGIC TRONG HANDLER (Invoke Phase)
    # Tái sử dụng đối tượng 'table' từ không gian bộ nhớ toàn cục
    try:
        payload = json.loads(event.get('body', '{}'))
        user_id = payload.get('userId')

        response = table.get_item(Key={'userId': user_id})
        item = response.get('Item', {})

        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({'user': item})
        }
    except Exception as e:
        return {'statusCode': 500, 'body': json.dumps({'error': str(e)})}
```

---

### 4. Kết Luận & Tài Liệu Tham Khảo

Bằng cách di chuyển logic khởi tạo SDK và kết nối Database ra ngoài Handler, bạn có thể giảm độ trễ thực thi của Lambda từ vài trăm milliseconds xuống chỉ còn vài milliseconds cho các lượt gọi tiếp theo.

**Tài liệu tham khảo từ AWS Blogs:**

- AWS Compute Blog: [Operating Lambda Performance Optimization (Part 1)](https://aws.amazon.com/vi/blogs/compute/operating-lambda-performance-optimization-part-1/)
- AWS Compute Blog: [Operating Lambda Performance Optimization (Part 2)](https://aws.amazon.com/vi/blogs/compute/operating-lambda-performance-optimization-part-2/)
