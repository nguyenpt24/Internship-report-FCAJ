---
title: "Blogs Posted"
date: 2026-08-04
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

# Technical Blogs Overview

This section summarizes technical articles authored and published on the [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj) community during the internship program:

---

### 1. [Blog 1 - Performance Optimization & Cold Start Handling in AWS Lambda](3.1-Blog1/)
This article explores Serverless Execution Environments, analyzing the 3-phase lifecycle (Init Phase, Invoke Phase, Shutdown Phase) of AWS Lambda and the root causes of Cold Start latencies. It presents practical Execution Context Reuse patterns by instantiating SDK/Database clients in the Global Scope to reduce execution latency from hundreds of milliseconds to single-digit milliseconds for warm invocations.

---

### 2. [Blog 2 - Single-Table Design Pattern in Amazon DynamoDB](3.2-Blog2/)
This article contrasts relational database (RDBMS) patterns with NoSQL modeling in Amazon DynamoDB. It details the Single-Table Design methodology using generic partition and sort keys (PK/SK Key Overloading), Item Collections, and Global Secondary Indexes (GSIs) to optimize Read/Write Capacity Unit (RCU/WCU) costs while ensuring sub-10ms response latencies at scale.

---

### 3. [Blog 3 - Building Protection & Traffic Control Layer with Amazon API Gateway](3.3-Blog3/)
This article examines Amazon API Gateway's strategic role as a centralized reverse proxy protecting Serverless backend services from DDoS threats and traffic surges. It highlights 4 core technology pillars: Token Bucket Rate Limiting & Throttling, Centralized Token Authentication via Lambda Authorizers, Response Caching, and Web Application Firewall (AWS WAF) integration.