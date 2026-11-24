---
tags: [aws, cloud, saa-c03]
service: "IAM (Identity and Access Management)"
category: "Security"
date: "2026-11-24"
---

# 02-IAM-Security

## 1. Executive Summary (What is it & Why use it?)
- **One-Sentence Definition:** **IAM (Identity and Access Management)** is a **Global Service** used to securely control authentication (who can sign in) and authorization (who has permissions to use resources) across AWS.
- **Primary Use Case:** Create and manage AWS users, groups, and roles, and control their access to AWS services and resources using granular permissions.
- **Scope / Responsibility Model:** **Global** (Security *IN* the Cloud — managed by the Customer).

---

## 2. Key Concepts & Architecture
- **Root Account:** Created by default when opening an AWS account. It **should never be used or shared** for daily tasks or applications—only for initial account setup and specific account-level tasks. Create individual IAM users instead.
- **Users:** Represent specific people or applications within your organization that need access to AWS.
  - A user **does not have to belong** to any group.
  - A user **can belong to multiple groups** simultaneously (inheriting the combined permissions of all those groups).
- **Groups:** Collections of IAM users used to assign permissions in bulk.
  - **Key Exam Rule:** Groups **can only contain users, never other groups** (no nested groups).
- **IAM Policies (JSON):** JSON documents attached to users, groups, or roles that explicitly define what actions are allowed or denied on specific AWS resources.
  - **Least Privilege Principle:** Never grant more permissions than a user strictly needs to perform their job.

### IAM Policy JSON Structure (Sample)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2ReadOnly",
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*"
      ],
      "Resource": "*"
    }
  ]
}
```
- **`Version`:** Policy language version (always `"2012-10-17"` in current AWS policies).
- **`Statement`:** One or more individual permission blocks.
- **`Sid` (Optional):** Statement ID (human-readable label).
- **`Effect`:** Only two valid values exist: `"Allow"` or `"Deny"`.
- **`Action`:** List of API calls allowed or denied (`service:ActionName`, supports wildcards like `*`).
- **`Resource`:** Specific AWS resources (identified by ARN) or `"*"` for all resources.

> **Cross-Platform Context (AWS vs. Windows / On-Premises):**
> This JSON policy schema (`Version`, `Statement`, `Effect`, `Action`, `Resource`) is **specific to AWS IAM** (and AWS-compatible APIs like MinIO or LocalStack). In **Windows Server / Active Directory**, permissions are not defined via JSON files—instead, access is enforced through **NTFS/Share ACLs** (Access Control Lists) and **GPOs (Group Policy Objects)**, while modern container/cloud platforms like Kubernetes use **YAML-based RBAC**.

---

## 3. CLI Commands / LocalStack Lab
```bash
# Example creation / query commands using awslocal
awslocal iam list-users
```

---

## 4. Security, IAM & Key Exam Limits
- **Required Permissions:** `iam:*` (Administrator) or granular `iam:CreateUser`, `iam:AttachGroupPolicy`, etc.
- **Critical Exam / Architecture Points:**
  - **Implicit Deny by Default:** Every new IAM user starts with zero permissions until explicitly granted via a policy.
  - **Explicit Deny Always Wins:** If a user inherits an `"Allow"` from one group and an `"Deny"` from another, AWS always denies the action.
  - **No Nested Groups:** Groups cannot contain other groups.

---

## 5. Quick Comparison (Key Differences vs. Similar Services)
| Feature | IAM User | IAM Group | IAM Role |
| :--- | :--- | :--- | :--- |
| **Target Entity** | Individual person or app | Collection of IAM users | AWS services (EC2, Lambda) or federated users |
| **Credentials** | Long-term (Password / Access Keys) | None (inherits to members) | Short-term temporary tokens (via STS) |
| **Ideal Use Case** | Human admins / developers | Role-based team permissions (Devs, Ops) | Allowing an EC2 instance to access S3 securely |

