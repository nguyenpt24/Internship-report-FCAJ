---
title: "Worklog Week 3"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

- Understand Amazon RDS relational databases and Amazon DynamoDB NoSQL databases.
- Learn in-memory caching with ElastiCache and domain management with Amazon Route 53.
- Initialize the DynamoDB table `WebSocketConnections` to store active connection IDs for the **Serverless Real-Time Chat App**.

### Tasks to be implemented this week:

| Day | Task | Start Date | Completion Date | Resource |
| --- | --- | --- | --- | --- |
| Mon | - Learn & create Amazon Relational Database Service (RDS) <br> - Configure Multi-AZ, Read Replicas & connect from EC2 | 06/22/2026 | 06/22/2026 | <https://cloudjourney.awsstudygroup.com/> |
| Tue | - NoSQL databases with Amazon DynamoDB <br> - **Practice creating `WebSocketConnections` Table**: Configure Primary Key `connectionId` (String), Pay-per-request billing | 06/23/2026 | 06/23/2026 | <https://cloudjourney.awsstudygroup.com/> |
| Wed | - In-memory caching with ElastiCache (Redis/Memcached) <br> - Practice AWS CLI queries against DynamoDB tables | 06/24/2026 | 06/24/2026 | <https://cloudjourney.awsstudygroup.com/> |
| Thu | - DNS management with Amazon Route 53 <br> - Create Hosted Zones, configure A and CNAME records pointing to EC2 / S3 | 06/25/2026 | 06/25/2026 | <https://cloudjourney.awsstudygroup.com/> |
| Fri | - **Practice:** Application scaling with EC2 Auto Scaling <br> - Connect Auto Scaling Groups with Load Balancers & setup CloudWatch monitoring | 06/26/2026 | 06/26/2026 | <https://cloudjourney.awsstudygroup.com/> |

### Week 3 Achievements:

- Successfully created DynamoDB table `WebSocketConnections` with `connectionId` as Partition Key, ready for managing Chat app WebSocket connection IDs.

- Mastered CRUD operations on DynamoDB using AWS Console and AWS CLI SDK.

- Understood low-latency NoSQL database design principles for real-time chat applications.
