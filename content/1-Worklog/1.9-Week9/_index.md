---
title: "Worklog Week 9"
date: 2026-08-03
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives (08/03/2026 - 08/04/2026):

- Complete Backend Lambda Handlers (`onConnect`, `onDisconnect`, `sendMessage`), configure WebSocket API Gateway, and integrate Web Frontend.
- Conduct End-to-End real-time message testing across 2 separate browser tabs for the **Serverless Real-Time Chat App**.
- Inspect CloudWatch Logs/Metrics, configure automated **CloudWatch Alarms**, and audit against AWS Well-Architected principles.
- Author resource cleanup scripts, finalize all report sections, and **officially submit the internship report**.

### Tasks to be implemented this week:

| Day | Task                                                                                                                                                                                                                                                                                                                                          | Start Date | Completion Date | Resource                                  |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | --------------- | ----------------------------------------- |
| Mon | - **Backend & API Gateway Programming**: Author 3 Node.js Lambda functions (`onConnect`, `onDisconnect`, `sendMessage`), create WebSocket API Gateway & connect Web Frontend (`index.html`) <br> - **End-to-End Chat App Testing**: Test 2 independent browser tabs sending real-time messages with < 100ms latency & test connection cleanup | 08/03/2026 | 08/03/2026      | <https://cloudjourney.awsstudygroup.com/> |
| Tue | - **Monitoring & Architecture Audit**: Inspect CloudWatch Logs Group `/aws/lambda/sendMessage`, configure CloudWatch Alarm `HighLambdaErrorAlert` & audit AWS Well-Architected Framework <br> - **Finalize & Submit Internship Report**: Write CLI Cleanup script, review all Hugo pages, and **officially submit the internship report**     | 08/04/2026 | 08/04/2026      | <https://cloudjourney.awsstudygroup.com/> |

### Week 9 Achievements:

- Successfully authored and deployed the **Serverless Real-Time Chat App** running live on AWS infrastructure with instant response times (< 100ms).

- Configured centralized CloudWatch monitoring, automated error notifications via CloudWatch Alarms, and security via Amazon Cognito User Pool.

- Conducted AWS Well-Architected Framework audit and authored safe CLI resource cleanup scripts.

- Completed the entire internship report on the Hugo platform.
