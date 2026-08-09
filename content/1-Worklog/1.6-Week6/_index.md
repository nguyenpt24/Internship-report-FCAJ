---
title: "Worklog Week 6"
date: 2026-07-13
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

- Securely manage remote server operations without open SSH/RDP ports using AWS Systems Manager.
- Build advanced monitoring dashboards with CloudWatch, Grafana, and VPC Flow Logs.
- Initialize Amazon Cognito User Pool to authenticate users logging into the **Serverless Real-Time Chat App**.

### Tasks to be implemented this week:

| Day | Task | Start Date | Completion Date | Resource |
| --- | --- | --- | --- | --- |
| Mon | - System management with AWS Systems Manager (SSM) <br> - Secure remote server access via Systems Manager Session Manager | 07/13/2026 | 07/13/2026 | <https://cloudjourney.awsstudygroup.com/> |
| Tue | - Advanced monitoring with CloudWatch & Grafana <br> - Advanced CloudWatch Workshop, VPC Flow Logs & Tags | 07/14/2026 | 07/14/2026 | <https://cloudjourney.awsstudygroup.com/> |
| Wed | - Access control security with IAM Permission Boundaries <br> - Advanced IAM Policies configuration for Chat project Lambda functions & VPC Endpoints | 07/15/2026 | 07/15/2026 | <https://cloudjourney.awsstudygroup.com/> |
| Thu | - Web application security with AWS WAF & encryption key management with AWS KMS <br> - Secure credential management with AWS Secrets Manager | 07/16/2026 | 07/16/2026 | <https://cloudjourney.awsstudygroup.com/> |
| Fri | - **Security Practice:** Create Amazon Cognito User Pool & App Client for Chat project <br> - Configure JWT Token verification to authorize Chat application access | 07/17/2026 | 07/17/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 6 Achievements:

- Successfully created Amazon Cognito User Pool & App Client to manage sign-up, sign-in, and issue JWT tokens for Chat App users.

- Configured IAM Least Privilege policies restricting Chat Lambda functions to access only the `WebSocketConnections` DynamoDB table.

- Mastered AWS Systems Manager operations and KMS data encryption security tools.
