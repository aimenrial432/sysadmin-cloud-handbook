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
- **User Access Methods & Credentials:**
  - **AWS Management Console (Web UI):** Authenticated via **Username + Password** (+ MFA). Managed via `create-login-profile`.
  - **Programmatic Access (CLI / SDK / Terraform):** Authenticated via **Access Key ID + Secret Access Key** (never meant to be memorized; automatically stored locally in `~/.aws/credentials` and `~/.aws/config`, safely outside project Git repositories).
  -
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
- **`Sid` (Optional):** Statement ID — a unique alphanumeric identifier within the same policy document used to label each individual statement.
- **`Effect`:** Only two valid values exist: `"Allow"` or `"Deny"`.
- **`Action`:** List of API calls allowed or denied (`service:ActionName`, supports wildcards like `*`).
- **`Resource`:** Specific AWS resources (identified by ARN) or `"*"` for all resources.

> **Cross-Platform Context (AWS vs. Windows / On-Premises):**
> This JSON policy schema (`Version`, `Statement`, `Effect`, `Action`, `Resource`) is **specific to AWS IAM** (and AWS-compatible APIs like MinIO or LocalStack). In **Windows Server / Active Directory**, permissions are not defined via JSON files—instead, access is enforced through **NTFS/Share ACLs** (Access Control Lists) and **GPOs (Group Policy Objects)**, while modern container/cloud platforms like Kubernetes use **YAML-based RBAC**.

---

## 3. CLI Commands / LocalStack Lab

> **Console vs. LocalStack Note:** Unlike the AWS Web Console wizard (which lets you tick managed policies during user creation), CLI/LocalStack workflows require creating the user/group first and explicitly attaching policies via ARN. *(Note: LocalStack Community simulates IAM state; real permission blocking requires `ENFORCE_IAM=1` in Pro).*

```powershell
# 1. Create IAM Group and User, then assign membership
awslocal iam create-group --group-name admin
awslocal iam create-user --user-name aimenrial432
awslocal iam add-user-to-group --user-name aimenrial432 --group-name admin

# 2. Attach an AWS Managed Policy (AdministratorAccess) to the group
awslocal iam attach-group-policy --group-name admin --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# 3. Create a Customer Managed Policy from a local JSON file (file must exist in current directory)
awslocal iam create-policy --policy-name EC2ReadOnlyCustom --policy-document file://policy.json

# 4. Set Console Password (Login Profile) & Generate Programmatic Access Keys
awslocal iam create-login-profile --user-name aimenrial432 --password "StrongPassword123!"
awslocal iam create-access-key --user-name aimenrial432

# 5. Configure AWS CLI Profiles & Switch Active Identity
aws configure                          # Sets permanent 'default' profile (replaces root by default)
aws configure --profile aimenrial432   # Stores credentials under a named profile in ~/.aws/credentials

# Switch profile temporarily for current PowerShell session:
$env:AWS_PROFILE = "aimenrial432"
Remove-Item Env:AWS_PROFILE            # Revert to default profile

# Verify active caller identity ("whoami" in AWS)
awslocal sts get-caller-identity
awslocal sts get-caller-identity --profile aimenrial432

# 6. Audit / Verification Commands
awslocal iam list-users
awslocal iam get-group --group-name admin
awslocal iam list-attached-group-policies --group-name admin
```

---

## 4. Security, IAM & Key Exam Limits
- **Required Permissions:** `iam:*` (Administrator) or granular `iam:CreateUser`, `iam:AttachGroupPolicy`, etc.
- **Policy Evaluation Logic (Strict Order):**
  1. **Explicit Deny:** Is there any applicable `"Effect": "Deny"`? $\rightarrow$ **DENIED** (evaluation stops immediately; overrides any `Allow`).
  2. **Explicit Allow:** Is there an applicable `"Effect": "Allow"`? $\rightarrow$ **ALLOWED**.
  3. **Implicit Deny (Default):** If neither exists $\rightarrow$ **DENIED** by default (every new IAM user starts with zero permissions).
- **Critical Exam / Architecture Points:**
  - **No Initial Deny Policy Needed:** Because of *Implicit Deny*, you do not attach a "deny all" policy when creating users—you start from zero and attach only the required `Allow` policies (*Least Privilege*), reserving `Explicit Deny` for strict guardrails/exceptions.
  - **No Nested Groups:** Groups cannot contain other groups.
  - **Access Key Limits & Rotation:** Maximum **2 active Access Keys** per IAM user at any time to allow seamless key rotation. `SecretAccessKey` is shown **only once** upon creation; if lost, delete the key and generate a new one. Never commit `.env` or credential files to Git.

---

## 5. Quick Comparison (Key Differences vs. Similar Services)
| Feature | IAM User | IAM Group | IAM Role |
| :--- | :--- | :--- | :--- |
| **Target Entity** | Individual person or app | Collection of IAM users | AWS services (EC2, Lambda) or federated users |
| **Credentials** | Long-term (Password / Access Keys) | None (inherits to members) | Short-term temporary tokens (via STS) |
| **Ideal Use Case** | Human admins / developers | Role-based team permissions (Devs, Ops) | Allowing an EC2 instance to access S3 securely |


Tags: Son utiles en muchos recursos para darle nombre, como por ejemplo indicar que un usuario pertenece a la tag ingenieria 