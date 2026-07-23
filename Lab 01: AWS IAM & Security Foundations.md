# AWS IAM & Security Foundations

**Difficulty:** Intermediate  
**Duration:** 2–3 hours  
**Tools:** AWS Console, AWS CLI, IAM

---

## Objective

This lab aims to provide a comprehensive understanding of **AWS Identity and Access Management (IAM)**, which is critical for securing AWS environments. Participants will learn to:

- Create and manage users, groups, roles, and policies
- Enforce Multi-Factor Authentication (MFA)
- Apply the principle of least privilege
- Use AWS CLI for policy management and auditing

**Note:** These skills are essential for Security & Compliance roles and are aligned with industry job descriptions.

---

## Tasks Overview

### Task 1.1: Create IAM User & Group (AWS Console)

**Purpose:** Establish a new IAM user with access to the AWS Management Console and attach appropriate permissions.

#### Steps:

1. **Navigate to IAM Users Creation**
   - AWS Console → IAM → Users → **Create User**

2. **Configure User Details**
   - Enter username: `devops-engineer-01`
   - Select **Provide user access to the AWS Management Console**

3. **Set Permissions**
   - Choose **Create an IAM user**
   - Set a custom password
   - Proceed to the next step

4. **Assign User to Group**
   - Select **Add user to group**
   
5. **Create New Group**
   - Click **Create Group**
   - Name: `DevOpsTeam`
   - Attach Policy: `AdministratorAccess` *(for lab purposes only; in production, always follow the least-privilege principle)*

6. **Complete Creation**
   - Finish setup
   - Download and securely save the `.csv` credentials file
   - Log in with the new IAM user using the provided sign-in URL

---

### Task 1.2: Enable MFA for IAM User

**Purpose:** Enhance security by enabling Multi-Factor Authentication.

#### Steps:

1. **Access User Security Settings**
   - IAM → Users → Select `devops-engineer-01` → **Security credentials** tab

2. **Assign MFA Device**
   - Under **Multi-factor authentication (MFA)** → **Assign MFA device**
   - Select **Authenticator app**
   - Click **Next**

3. **Configure MFA App**
   - Open **Google Authenticator** or **Authy** on your phone
   - Scan the QR code displayed

4. **Verify MFA**
   - Enter two consecutive 6-digit codes from the app
   - Click **Add MFA**

5. **Test MFA**
   - Sign out and sign back in
   - Confirm the MFA prompt appears during login

---

### Task 1.3: Create IAM Role & Attach to EC2

**Purpose:** Grant EC2 instances read-only access to S3 buckets through an IAM role.

#### Steps:

1. **Create IAM Role**
   - IAM → Roles → **Create Role**
   - Trusted entity: **AWS Service**
   - Use case: **EC2**

2. **Attach Policies**
   - Attach policy: `AmazonS3ReadOnlyAccess`
   - Name role: `EC2-S3-ReadOnly-Role`

3. **Attach Role to EC2 Instance**
   - EC2 → Select a running instance → **Actions** → **Security** → **Modify IAM Role**
   - Attach `EC2-S3-ReadOnly-Role`
   - Save changes

4. **Test Role**
   - SSH into the EC2 instance
   - Run:
   
     ```bash
     aws s3 ls
     ```
   - The command should list S3 buckets without requiring manual credential configuration

---

### Task 1.4: Create & Test Custom IAM Policy (via AWS CLI)

**Purpose:** Define a custom policy for specific permissions and attach it to the user.

#### Steps:

1. **Create Policy JSON File**

Create a file named `devops-policy.json` with the following content:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-devops-bucket",
        "arn:aws:s3:::my-devops-bucket/*"
      ]
    }
  ]
}
```

2. **Create the Policy in AWS**

```bash
aws iam create-policy \
  --policy-name DevOpsCustomPolicy \
  --policy-document file://devops-policy.json \
  --region ap-south-1
```

3. **Attach Policy to User**

Replace `ACCOUNT_ID` with your AWS account ID:

```bash
aws iam attach-user-policy \
  --user-name devops-engineer-01 \
  --policy-arn arn:aws:iam::ACCOUNT_ID:policy/DevOpsCustomPolicy
```

4. **Simulate Policy for Testing**

```bash
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::ACCOUNT_ID:user/devops-engineer-01 \
  --action-names ec2:StopInstances s3:PutObject
```

---

## Best Practices & Pro Tips

### **Best Practice: Least Privilege Principle**

- Never attach `AdministratorAccess` in production environments.
- Always create and assign custom policies that grant only necessary permissions.
- Use **IAM Access Analyzer** to identify unused permissions.
- Rotate access keys every 90 days.
- Avoid hard-coding credentials; use IAM Roles for EC2, ECS, Lambda.
- Enable **AWS CloudTrail** to audit all IAM API calls.

### **Pro Tip: Account Auditing & Resource Tagging**

- Use `aws iam get-account-authorization-details` to audit the entire account.
- Tag IAM resources with `Team`, `Project`, and `Environment` for effective cost attribution and governance.

---

This manual provides a structured walk-through of essential IAM security practices on AWS, suitable for onboarding new team members or reinforcing security protocols in your environment.

---
