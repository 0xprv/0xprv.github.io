---
layout: post
title: "Realtime Monitoring of Bank Transactions using AI/ML (Nigeria)"
date: 2025-04-01
categories: [projects, fintech, ai]
tags: [java, springboot, rust, python, typescript, machine-learning, fraud-detection]
---

# Bank Surveillance & Monitoring ML Based Software For Nigeria - Itrack

**Published:** February 1, 2025

---

## Overview

This project was created under an NDA signed with Itrack, a company based in Nigeria. I led the development of an enterprise software system designed to track live transactions directly from the bank's servers. The system applies various machine learning algorithms, clustering techniques, and deterministic rules to flag suspicious transactions based on predefined conditions.

Once flagged, transactions are grouped into cases, which are then automatically assigned to designated personnel based on their respective bank branches. These cases are sent to the assigned individuals via email, allowing them to review and take action accordingly.

![Itrack Main Dashboard](/assets/main.png)

---

## Tech Stack

**Languages & Frameworks:**
- **JAVA** (Spring Boot Backend)
- **RUST** (For librdkafka & Service Running in backend for monitoring)
- **TypeScript/JavaScript** (React)
- **Python** (ML & data processing)

**Technologies:**
- **KAFKA** (Real-time transaction streaming)
- **JPA Repository** (Database ORM)
- **GO Rule** (Rule engine)
- **REDIS** (Profile caching)

**Approximate Lines of Code:** ~25,000

---

## Deployment

**Platform:** Ubuntu Server 24.04.2 LTS
**Tested on:** 4GB RAM for efficiency

![System Resource Usage (htop)](/assets/htop.png)

The use of Rust helps significantly with resource efficiency, as shown in the performance metrics above.

---

## Backend Tech (JAVA)

The backend of the software, which handles four different types of databases, is entirely developed in Java using the Spring Boot framework with JPA. The system is designed to support dynamic switching between databases—**PostgreSQL, MySQL, MS SQL Server, and Oracle**—based on user selection during the project setup or later through the software's settings.

### Database Configuration

Users can choose their preferred database either during the initial setup or by changing the configuration from within the software settings at any time.

![Initial Database Setup](/assets/initdb.png)

![Change Database Configuration](/assets/changedb.png)

### Architecture

I chose Java for this project not only because it's well-suited for enterprise-level software, but also because it was the preferred choice of the client, Itrack. I developed:

- **37 model classes** with corresponding controller, service, and repository layers
- **148 Java files** purely dedicated to the backend
- All following the standard Spring Boot JPA structure

### Email Notifications

I also used **JavaMail** for handling automated email notifications. Users can configure their email settings within the application, enabling the system to automatically send emails based on certain events—such as notifying relevant personnel when a case is assigned for reviewing suspicious transactions.

![Email Configuration](/assets/emailsetting.png)

---

## REDIS (Profile Caching)

In Itrack, I integrated REDIS storage hosted locally to store different profiles performing in transactions. Profiles are the atomic entities here—every transaction has associated profiles and individuals involved in it.

Redis holds only the profiles for ultra-fast retrieval speeds. This section demonstrates various profiles, each currently managing between **20,000 to 35,000 accounts** and associated data.

### Performance Metrics

- **Current load time:** 3–4 seconds (optimizing to under 2 seconds)
- **Daily transaction volume:** ~100,000 accounts
- **Storage size:** ~200-300 MB

When new transactions enter the system, it checks these profile tables—if the account exists, it updates the information; if not, a new account is automatically created.

![Profile Management Demo](/assets/profiles.mp4)

---

## RUST (Handling Kafka Streaming)

To enhance the speed and memory efficiency of the system—particularly for handling a high volume of real-time transactions streaming through Kafka—I integrated Rust alongside the Java backend.

### Why Rust?

Rust was specifically chosen for its:
- **Performance benefits** - Near-C level speed for real-time processing
- **Memory safety** - Statically-typed system with low-level control
- **Scalability** - Can handle high-throughput streams efficiently

### Implementation

I utilized the **[Librdkafka Rust wrapper](https://github.com/confluentinc/librdkafka)** (originally written in C) to interface with Apache Kafka. This allowed the system to handle transaction streams at a **rate of 100–200 transactions per second**—significantly outperforming Java, which could only reliably handle around 1–2 transactions per second under similar conditions.

### Background Services

In addition to processing Kafka topics, Rust also powers **3–4 background Linux services running 24/7**. These services are responsible for:

1. **Non-realtime rule evaluation**
2. **Fetching and processing transactions from Kafka logs**
3. **Velocity checks** on transactions (e.g., flagging sudden spikes in transaction volume or abnormal behavioral patterns)

Once suspicious activity is detected, the Rust services create appropriate cases in the backend and trigger email notifications to the assigned users for review.

---

## Python (ML Component)

Python is primarily used in this project for implementing the machine learning components, thanks to its simplicity and strong ecosystem in the data science domain.

### Optimization

A dedicated Linux service runs Python scripts that load transactional data into Pandas for processing. To improve performance with large datasets, I leveraged:
- **Numba** - JIT compilation for numerical operations
- **Cython** - C-level performance for data transformation

### ML Algorithms

The system uses **four types of traditional machine learning algorithms** to detect anomalies in transaction behavior:

1. **Isolation Forest**
2. **One-Class SVM**
3. **Clustering**
4. **Decision Tree**

Deep learning approaches were intentionally avoided due to limited resources and insufficient data for effective training. Instead, we focused on robust, interpretable, and computationally efficient traditional ML algorithms that work well at scale.

### Transaction Categories

These models are applied independently to four types of transactions:

1. **Inward Transactions**
2. **Outward Transactions**
3. **Branch Transactions**
4. **Card Transactions**

Each category is processed in its own thread or process, allowing the system to parallelize computation. The total volume handled is approximately **100,000 transactions per category**. Once anomalies are detected, results are sent to the frontend via APIs, where they're displayed in real-time on a dashboard.

---

## Rules Engine

The software includes a comprehensive rule engine where users can create different conditional queries and rules. These rules are applied to live Kafka stream transactions, and if any transaction is flagged according to a rule, an alert and a case are automatically triggered. The case is then automatically assigned to an available bank staff member for further investigation.

![Rule Engine Demo](/assets/rule_demo.mp4)

### Rule Types

#### 1. Real-time Rules

Real-time rules are triggered at the exact moment a transaction occurs. As soon as a new Kafka log stream is received, the rules are immediately applied based on the transaction details. If any condition matches, the transaction is flagged, and a case is automatically generated.

#### 2. Non-Real-time Rules

Non-real-time rules work differently from real-time ones. Instead of processing transactions instantly one by one, these rules handle transactions in bulk. Each non-real-time rule is configured with an **interval**—such as every few seconds or minutes—defining how often it should run.

**Example:** If a rule has a 5-minute interval, then all transactions received during that 5-minute window are processed together. If out of 1,000 transactions, 200 meet the rule's conditions, those 200 flagged transactions are grouped, and a single case is created for them.

Since these rules are more resource-intensive, they are executed in the background using Rust threads for better performance and parallel processing.

![Rule Explanation Diagram](/assets/rule_expl.png)

---

## User Management Module

This module is dedicated to user management. Each user is assigned specific permissions, which determine what features or sections of the software they can access.

Permissions can be based on roles such as **admin**, or on organizational structures like **branch-based** or **region-based** access. This ensures users only interact with the parts of the system that are relevant to their responsibilities and access level.

![User Management Demo](/assets/user_demo.mp4)

---

## Other Notable Features

While this software has many features, due to the NDA signed with the client, I can only show some of them. Here are the key features:

### Workflow Configuration

Workflow defines the process for handling email notifications during transaction processing and rule triggering. It allows configuration of which users or officers should receive emails, including options for **To, CC, and BCC**.

![Workflow Demo](/assets/workflow.mp4)

### Branches

Branches are entities in the software that define different bank branches to help distinguish which software instance should send emails and to which branch they are associated.

![Branches Demo](/assets/branches.mp4)

### Regions

Regions represent specific areas or locations where branches are situated, allowing the classification of data based on different bank regions.

### Roles & Permissions

This software allows assigning specific roles and permissions to individual users, ensuring they can only access the areas they are authorized for, while restricting access to other parts of the system.

![Roles & Permissions Demo](/assets/role.mp4)

### Global Audit/Log

The global logs record all different actions happening on the software—from creation to deletion of entities. It records every possible CRUD operation and user interaction for auditing purposes by administrators.

![Global Audit Log Demo](/assets/globallog.mp4)

### Custom Email Templates

In this software, users can create custom HTML email templates and configure them through the UI. This allows emails sent by the system to match the look and content based on the specific task being performed.

![Email Templates Demo](/assets/template.mp4)

### Dashboards

You can view overall performance data and track the operations performed by the software, including how much data has been processed, all within interactive dashboards.

![Dashboard 1](/assets/dash1.png)

![Dashboard 2](/assets/dash2.png)

![Dashboard 3](/assets/dash3.png)

### Case Management

This section serves as the final screen displaying processed information for flagged transactions identified by the rule engine. It highlights flagged activity, creates corresponding cases, and assigns them to specific users within the software for resolution.

![Case Management Demo](/assets/case.mp4)

---

## Conclusion

The Itrack project demonstrates a comprehensive approach to building enterprise-grade banking software with advanced anomaly detection capabilities. By strategically combining multiple technologies—Java for robust backend services, Rust for high-performance transaction processing, Python for machine learning, and a modern React frontend—the system achieves both scalability and efficiency while maintaining security and auditability standards required in the financial sector.

**Note:** This project was developed under NDA. Some features and specific implementation details have been redacted to protect client confidentiality.
