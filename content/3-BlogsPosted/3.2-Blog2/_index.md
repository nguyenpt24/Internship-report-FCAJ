---
title: "Blog 2: Single-Table Design in Amazon DynamoDB"
date: 2026-08-04
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Single-Table Design Pattern in Amazon DynamoDB: Optimizing Performance & Cost for Serverless Systems

---

In modern cloud application development, **Amazon DynamoDB** is renowned as a fully managed NoSQL database service capable of maintaining single-digit millisecond response latencies at any scale — whether storing a few gigabytes or hundreds of terabytes.

However, many developers transitioning from Relational Database Management Systems (RDBMS) to DynamoDB fall into a common pitfall: applying Multi-Table normalized schemas directly to NoSQL. This article explores the **Single-Table Design** pattern — an ultimate data modeling technique to unlock DynamoDB's full potential.

---

### 1. Context: Core Differences Between RDBMS and DynamoDB

To understand why Single-Table Design emerged, we must contrast operational models between the two database paradigms:

- **RDBMS Model (MySQL, PostgreSQL)**: Optimized for disk storage costs. Data is normalized across separate tables (`Users`, `Orders`, `OrderItems`). Fetching a user's complete order history requires relational `JOIN` operations. As datasets grow to millions of rows, `JOIN` queries consume significant CPU scanning disk indexes, creating performance bottlenecks and latency spikes.
- **DynamoDB Model (NoSQL)**: Optimized for query latency & compute efficiency. DynamoDB does not support `JOIN` operations. All data required for a single application screen should be retrieved in a single read request.

Therefore, the golden rule of DynamoDB modeling is: **You must define all application Access Patterns before designing table schemas.**

---

### 2. Core Principles of Single-Table Design

The Single-Table Design pattern stores all related entity types (e.g., User Profiles, Orders, Order Items, Invoices) within a single DynamoDB table.

#### How Generic Keys Work

Instead of fixed primary key column names like `UserId` or `OrderId`, a Single-Table Design uses generic key names:

- **Partition Key (PK)**: Determines physical data partition placement.
- **Sort Key (SK)**: Sorts items within a partition and categorizes entity types.

By combining string prefixes/namespaces with delimiters (such as `#`), we perform **Key Overloading**. For example:

- **User Profile Item**: Set `PK = USER#1001` and `SK = METADATA#1001`.
- **User Order Item**: Set `PK = USER#1001` and `SK = ORDER#2026#001`.
- **Order Line Item**: Set `PK = ORDER#2026#001` and `SK = ITEM#SKU-992`.

#### The Item Collections Concept

All items sharing the same `PK = USER#1001` reside on the same physical partition, forming an **Item Collection**.

When displaying a user profile alongside order history, the application issues a single `Query` request with `PK = USER#1001`. DynamoDB returns both profile metadata and order lists in a single round-trip taking under 10 milliseconds.

---

### 3. Bidirectional Access via Global Secondary Indexes (GSI)

A common challenge in Single-Table Design is reverse querying: (e.g., given `OrderId = ORDER#2026#001`, finding which `UserId` owns it).

The solution is deploying a **Global Secondary Index (GSI)**:

- Create index **GSI-1** with inverted key attributes: `GSI1-PK = SK` and `GSI1-SK = PK`.
- Query GSI-1 with `GSI1-PK = ORDER#2026#001` to instantly identify the owner account.

GSIs asynchronously replicate data in real time, supporting secondary access patterns without impacting primary table performance.

---

### 4. Key Benefits of Single-Table Design

- **Cost Savings (RCU / WCU)**: In DynamoDB, pricing depends on Read/Write Capacity Units. Consolidating entities into a single table simplifies Auto-scaling and capacity provisioning, eliminating wasted capacity across idle tables.
- **Optimal Retrieval Speed**: Minimizes network round-trips by retrieving complex hierarchical data structures in one request.
- **Seamless Scalability**: A single table scales effortlessly from thousands to millions of requests per second without infrastructure redesign.

---

### 5. When to Use (and When NOT to Use) Single-Table Design

#### Use When:

- Medium-to-large Serverless applications with well-defined Access Patterns identified during design.
- High-throughput, latency-sensitive systems (e.g., e-commerce, financial payments, online gaming).
- Maximizing AWS infrastructure cost efficiency.

#### Avoid When:

- Systems requiring ad-hoc, flexible queries or unconstrained Business Intelligence (BI) analytics.
- Engineering teams new to NoSQL (starting with a simpler Multi-Table model prevents initial key design mistakes).

---

### Conclusion & References

Transitioning from multi-table RDBMS patterns to DynamoDB Single-Table Design requires a fundamental mindset shift. By partitioning entities via `PK` and categorizing them via `SK`, applications gain a high-performance database foundation built for extreme scale.

**Official References from AWS:**

- Amazon DynamoDB Developer Guide: [Best Practices for Designing DynamoDB Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
