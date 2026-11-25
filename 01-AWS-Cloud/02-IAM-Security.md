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
- **Account Alias & Sign-in URL:**
  - By default, IAM users sign in to the AWS Web Console via a URL containing the **12-digit Account ID** (`https://<12-digit-account-id>.signin.aws.amazon.com/console`).
  - An **Account Alias** replaces that 12-digit ID with a globally unique, friendly name (`https://<alias>.signin.aws.amazon.com/console`).
- **Users:** Represent specific people or applications within your organization that need access to AWS.
  - A user **does not have to belong** to any group.
  - A user **can belong to multiple groups** simultaneously (inheriting the combined permissions of all those groups).
  - **Policy Inheritance & Effective Permissions:** A user's effective permissions are the union (cumulative sum) of all policies attached to every group they belong to, plus any direct user policies.
    - **AWS Managed Policies:** Pre-built, read-only policies created and maintained by AWS (e.g., `AdministratorAccess`, `IAMReadOnlyAccess`, `AmazonS3ReadOnlyAccess`).
    - **Customer Managed Policies:** Custom standalone JSON policies created and maintained by you; reusable across multiple users, groups, or roles with version history (`v1`, `v2`, etc.).
    - **Inline Policies:** Policies embedded directly into a single user, group, or role—strictly 1-to-1 (deleted automatically if the parent identity is deleted). Useful only for one-off exceptions; Managed Policies are preferred.
- **Groups:** Collections of IAM users used to assign permissions in bulk.
  - **Key Rule:** Groups **can only contain users, never other groups** (no nested groups).
- **User Access Methods & Credentials:**
  - **AWS Management Console (Web UI):** Authenticated via **Account ID/Alias + IAM Username + Password** (+ MFA). Managed via `create-login-profile`.
  - **Programmatic Access (CLI / SDK / Terraform):** Authenticated via **Access Key ID + Secret Access Key** (never meant to be memorized; automatically stored locally in `~/.aws/credentials` and `~/.aws/config`, safely outside project Git repositories).
  - **Multi-Session Support (AWS Console):** Allows signing into up to **5 different IAM users or AWS accounts simultaneously** across tabs in the same browser without overwriting session cookies (useful for side-by-side permission testing across services like EC2 or S3).
- **IAM Tags (`Key: Value` Metadata):** Optional key-value pairs attached to IAM users or roles (e.g., `Department: Engineering`, `Environment: Prod`).
  - Used for organization, cost tracking, auditing, and **ABAC (Attribute-Based Access Control)**—granting dynamic access when a user's tag matches a resource's tag (`aws:PrincipalTag` vs. `aws:ResourceTag`).
- **IAM Policies (JSON):** JSON documents attached to users, groups, or roles that explicitly define what actions are allowed or denied on specific AWS resources.
  - **Least Privilege Principle:** Never grant more permissions than a user strictly needs to perform their job.

### IAM Policy JSON Structure (Sample)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2ReadOnlyWithMFA",
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*"
      ],
      "Resource": "*",
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```
- **`Version`:** Policy language version (always `"2012-10-17"` in current AWS policies).
- **`Statement`:** One or more individual permission blocks.
- **`Sid` (Optional):** Statement ID — a unique alphanumeric identifier within the same policy document used to label each individual statement.
- **`Principal` (Resource-Based Policies Only):** Specifies the account, user, role, or AWS service allowed/denied access (e.g., S3 Bucket Policies or Role Trust Policies). **Omitted** in identity-based policies (already implied by the attached user/group/role) and **cannot target an IAM Group**.
- **`Effect`:** Only two valid values exist: `"Allow"` or `"Deny"`.
- **`Action`:** List of API calls allowed or denied (`service:ActionName`). Supports prefix/full wildcards (`*`): e.g., `"Action": "*"` grants all AWS services/actions, while `"iam:Get*"` and `"iam:List*"` group all read/list API calls for IAM.
- **`Resource`:** Specific AWS resources (identified by ARN) or `"*"` for all resources.
- **`Condition` (Optional):** Extra rule block specifying *when* the statement takes effect (`Operator` + `Condition Key` + `Value`). The policy applies **only if** the condition evaluates to true (e.g., enforcing MFA via `aws:MultiFactorAuthPresent`, restricting by `aws:SourceIp`, requiring HTTPS via `aws:SecureTransport`, or matching resource tags).

> **Cross-Platform Context (AWS vs. Windows / On-Premises):**
> This JSON policy schema (`Version`, `Statement`, `Effect`, `Action`, `Resource`) is **specific to AWS IAM** (and AWS-compatible APIs like MinIO or LocalStack). In **Windows Server / Active Directory**, permissions are not defined via JSON files—instead, access is enforced through **NTFS/Share ACLs** (Access Control Lists) and **GPOs (Group Policy Objects)**, while modern container/cloud platforms like Kubernetes use **YAML-based RBAC**.

---

## 3. CLI Commands / LocalStack Lab

> **AWS Console Dashboard vs. LocalStack Environment Note:**
> - **Permission Assignment:** Unlike the AWS Web Console wizard (which lets you tick managed policies during user creation), CLI/LocalStack workflows require creating the user/group first and explicitly attaching policies via ARN.
> - **Console Sign-in & Alias:** In real AWS, IAM users log in through a dedicated web login portal using their Account ID/Alias, username, and password. LocalStack does **not** render an IAM web login form—its Resource Browser (`app.localstack.cloud`) connects directly to the local API (`localhost:4566`), so identity switching is performed via CLI profiles (`--profile` or `default`) while Account Aliases are managed via CLI. *(Note: LocalStack Community simulates IAM state; real permission blocking requires `ENFORCE_IAM=1` in Pro).*

```powershell
# 1. Create IAM Group and User, then assign membership
awslocal iam create-group --group-name admin
awslocal iam create-user --user-name aimenrial432
awslocal iam add-user-to-group --user-name aimenrial432 --group-name admin

# 2. Attach an AWS Managed Policy (AdministratorAccess) to the group
awslocal iam attach-group-policy --group-name admin --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# 3. Create a Customer Managed Policy from a local JSON file (file must exist in current directory)

#(Reusable standalone policy: MyIAMPermissions)
$policyJson = @'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:ListUsers",
        "iam:GetUser"
      ],
      "Resource": "*"
    }
  ]
}
'@

awslocal iam create-policy --policy-name MyIAMPermissions --policy-document $policyJson

# 4. Add and list metadata tags on an IAM user
awslocal iam tag-user --user-name aimenrial432 --tags Key=Department,Value=Engineering Key=Role,Value=CloudAdmin
awslocal iam list-user-tags --user-name aimenrial432

# 5. Manage Account Alias (replaces 12-digit Account ID on AWS Console sign-in URL)
awslocal iam create-account-alias --account-alias aimenrial-cloud-lab
awslocal iam list-account-aliases

# 6. Set Console Password (Login Profile) & Generate Programmatic Access Keys
awslocal iam create-login-profile --user-name aimenrial432 --password "StrongPassword123!"
awslocal iam create-access-key --user-name aimenrial432

# 7. Configure AWS CLI Profiles & Switch Active Identity
aws configure                          # Sets permanent 'default' profile (replaces root by default)
aws configure --profile aimenrial432   # Stores credentials under a named profile in ~/.aws/credentials

# Switch profile temporarily for current PowerShell session:
$env:AWS_PROFILE = "aimenrial432"
Remove-Item Env:AWS_PROFILE            # Revert to default profile

# Verify active caller identity ("whoami" in AWS)
awslocal sts get-caller-identity
awslocal sts get-caller-identity --profile aimenrial432

# 8. Audit / Verification Commands
awslocal iam list-users
awslocal iam get-group --group-name admin
awslocal iam list-attached-group-policies --group-name admin

# 9. Inspect AWS Managed Policies & View their JSON Definition (Built-in Policies)
awslocal iam list-policies --scope AWS --query "Policies[?contains(PolicyName, 'S3')].[PolicyName, Arn]" --output table
awslocal iam get-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
awslocal iam get-policy-version --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess --version-id v1

# 10. Inspect Direct User Policies & Cleanup Groups
awslocal iam list-attached-user-policies --user-name aimenrial432
awslocal iam detach-user-policy --user-name aimenrial432 --policy-arn arn:aws:iam::aws:policy/IAMReadOnlyAccess
awslocal iam delete-group --group-name developers

```

### Mini-Lab: Least Privilege & Multi-Profile Testing (Admin vs. Read-Only)
This lab simulates **Principle of Least Privilege** by creating a restricted `developers` group (`AmazonS3ReadOnlyAccess`) and switching between `aimenrial432` (Admin) and `dev-readonly` (Restricted) using named CLI profiles.

```powershell
# 1. Create a restricted group with S3 Read-Only permissions and assign a new user
awslocal iam create-group --group-name developers
awslocal iam attach-group-policy --group-name developers --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

awslocal iam create-user --user-name dev-readonly
awslocal iam add-user-to-group --user-name dev-readonly --group-name developers

# 2. Generate Access Keys for the restricted user and register a separate CLI profile
awslocal iam create-access-key --user-name dev-readonly
aws configure --profile dev-readonly

# 3. Verify identity switching between Admin (default) and Restricted user
awslocal sts get-caller-identity                         # Returns arn:aws:iam::000000000000:user/aimenrial432
awslocal sts get-caller-identity --profile dev-readonly  # Returns arn:aws:iam::000000000000:user/dev-readonly

# 4. Test S3 Actions across both identities
awslocal s3 mb s3://lab-iam-test-bucket                            # Admin creates bucket (Allowed)
awslocal s3 ls --profile dev-readonly                              # Read-only user lists buckets (Allowed)
awslocal s3 mb s3://unauthorized-bucket --profile dev-readonly     # Write attempt: AccessDenied in AWS / LocalStack ENFORCE_IAM=1

# 5. Cleanup test buckets
awslocal s3 rb s3://lab-iam-test-bucket
awslocal s3 rb s3://unauthorized-bucket
```

> **Lab Takeaway (LocalStack Community vs. Real AWS):**
> Even when `dev-readonly` signs requests via `--profile dev-readonly`, **LocalStack Community** logs the caller identity (`user/dev-readonly`) without blocking `s3:CreateBucket`. In **real AWS** (or LocalStack Pro with `ENFORCE_IAM=1`), step 4 fails with `An error occurred (AccessDenied) when calling the CreateBucket operation: Access Denied` because `AmazonS3ReadOnlyAccess` only grants `s3:Get*` and `s3:List*`.

---

## 4. Security, IAM & Key Exam Limits
- **Required Permissions:** `iam:*` (Administrator) or granular `iam:CreateUser`, `iam:AttachGroupPolicy`, etc.
- **Policy Evaluation Logic (Strict Order):**
  1. **Explicit Deny:** Is there any applicable `"Effect": "Deny"`? $\rightarrow$ **DENIED** (evaluation stops immediately; overrides any `Allow`).
  2. **Explicit Allow:** Is there an applicable `"Effect": "Allow"`? $\rightarrow$ **ALLOWED**.
  3. **Implicit Deny (Default):** If neither exists $\rightarrow$ **DENIED** by default (every new IAM user starts with zero permissions).
- **Critical Architecture & Exam Rules:**
  - **No Initial Deny Policy Needed:** Because of *Implicit Deny*, you do not attach a "deny all" policy when creating users—you start from zero and attach only the required `Allow` policies (*Least Privilege*), reserving `Explicit Deny` for strict guardrails/exceptions.
  - **No Nested Groups:** Groups cannot contain other groups.
  - **Account Alias Limits:** Only **1 Account Alias** per AWS account; the alias name must be **globally unique** across all AWS accounts (lowercase letters, numbers, and hyphens only).
  - **Access Key Limits & Rotation:** Maximum **2 active Access Keys** per IAM user at any time to allow seamless key rotation. `SecretAccessKey` is shown **only once** upon creation; if lost, delete the key and generate a new one. Never commit `.env` or credential files to Git.
  - **Console Multi-Session Limit:** The AWS Management Console supports up to **5 active concurrent sessions** per browser.
  - **Cross-Group Conflict Resolution:** If a user belongs to multiple groups (`dev` + `marketing`) and one group grants `Allow` while the other enforces an `Explicit Deny` on the same action, **`Explicit Deny` always wins**.

---

## 5. Quick Comparison (Key Differences vs. Similar Services)
| Feature            | IAM User                           | IAM Group                               | IAM Role                                       |
| :----------------- | :--------------------------------- | :-------------------------------------- | :--------------------------------------------- |
| **Target Entity**  | Individual person or app           | Collection of IAM users                 | AWS services (EC2, Lambda) or federated users  |
| **Credentials**    | Long-term (Password / Access Keys) | None (inherits to members)              | Short-term temporary tokens (via STS)          |
| **Ideal Use Case** | Human admins / developers          | Role-based team permissions (Devs, Ops) | Allowing an EC2 instance to access S3 securely |

