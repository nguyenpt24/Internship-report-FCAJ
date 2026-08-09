---
title: "Worklog Tuần 9"
date: 2026-08-03
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9 (03/08/2026 - 04/08/2026):

- Hoàn thiện lập trình Backend Lambda Handlers (`onConnect`, `onDisconnect`, `sendMessage`), cấu hình WebSocket API Gateway và kết nối Web Frontend.
- Thực hành kiểm thử End-to-End truyền nhận tin nhắn Real-Time trên 2 tab trình duyệt cho dự án **Serverless Real-Time Chat App**.
- Kiểm tra CloudWatch Logs/Metrics, cấu hình **CloudWatch Alarm** báo lỗi tự động & rà soát theo chuẩn AWS Well-Architected.
- Viết kịch bản dọn dẹp tài nguyên (Cleanup script), hoàn thiện toàn bộ bộ báo cáo thực tập và **nộp báo cáo chính thức**.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                                                                                                                                                                                                                                                                                                        | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------ | --------------- | ----------------------------------------- |
| 2   | - **Lập trình Backend & API Gateway**: Lập trình 3 hàm Lambda Node.js (`onConnect`, `onDisconnect`, `sendMessage`), tạo WebSocket API Gateway & ghép nối Web Frontend (`index.html`) <br> - **Kiểm thử End-to-End Chat App**: Mở 2 tab trình duyệt gửi tin nhắn real-time độ trễ < 100ms & thử nghiệm ngắt kết nối dọn dẹp connectionId          | 03/08/2026   | 03/08/2026      | <https://cloudjourney.awsstudygroup.com/> |
| 3   | - **Giám sát & Đánh giá Architecture**: Kiểm tra CloudWatch Logs Group `/aws/lambda/sendMessage`, cấu hình CloudWatch Alarm `HighLambdaErrorAlert` & rà soát chuẩn AWS Well-Architected Framework <br> - **Hoàn tất & Nộp Báo cáo Thực tập**: Viết kịch bản Cleanup script, kiểm tra toàn bộ website Hugo và **nộp báo cáo thực tập chính thức** | 04/08/2026   | 04/08/2026      | <https://cloudjourney.awsstudygroup.com/> |

### Kết quả đạt được tuần 9:

- Lập trình và triển khai thành công ứng dụng **Serverless Real-Time Chat App** chạy thực tế trên hạ tầng AWS với độ trễ phản hồi tức thì (< 100ms).

- Cấu hình đầy đủ hệ thống giám sát CloudWatch tập trung, bắt lỗi tự động qua CloudWatch Alarm và bảo mật bằng Amazon Cognito User Pool.

- Đánh giá kiến trúc đạt chuẩn AWS Well-Architected Framework và chuẩn bị kịch bản Cleanup tài nguyên an toàn.

- Hoàn thành toàn bộ bộ báo cáo thực tập trên website Hugo đúng hạn.
