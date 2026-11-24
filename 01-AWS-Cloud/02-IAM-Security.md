---
tags: [aws, cloud, saa-c03]
service: "02-IAM-Security"
category: "" # Compute | Storage | Networking | Security | Database
date: "2026-11-24"
---

# 02-IAM-Security

## 1. Executive Summary (What is it & Why use it?)
- **One-Sentence Definition:** IAM = Identity and Access Managment, Global Service
- **Primary Use Case:** Create users and assign them to a group
- **Scope / Responsibility Model:** (Regional / Global / Zonal)

## 2. Key Concepts & Architecture
- Root account created by default, shouldnt be used or shared. Only used for the first account creation. Create users instead.
- Users are people within your organization and can be grouped. User don´t have to belong a group, and can belong on multiple groups.
- Groups only can contain users, not other groups.

## 3. CLI Commands / LocalStack Lab
```bash
# Example creation / query commands using awslocal
awslocal ...
```

## 4. Security, IAM & Key Exam Limits
- **Required Permissions:** 
- **Critical Exam / Architecture Points:** 
  - 

## 5. Quick Comparison (Key Differences vs. Similar Services)
| Feature | 02-IAM-Security | Alternative |
| :--- | :--- | :--- |
| Latency / Performance | | |
| Persistence / Durability | | |
| Ideal Use Case | | |


IAM: Permissions
