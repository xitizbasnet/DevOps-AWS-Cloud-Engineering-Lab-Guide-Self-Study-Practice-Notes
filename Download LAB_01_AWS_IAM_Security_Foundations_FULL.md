# LAB 01: AWS IAM & Security Foundations

> **Difficulty:** Intermediate
>
> **Duration:** 2–3 Hours
>
> **Tools:** AWS Console, AWS CLI, IAM

## Objective

Understand and configure AWS Identity and Access Management (IAM), the backbone of every secure AWS environment. You will create users, groups, roles, and policies, enforce MFA, and apply least-privilege access, a mandatory skill under Security & Compliance.

---

# Task 1.1: Create IAM User & Group (AWS Console)

## Step 1
Navigate to:

`AWS Console → IAM → Users → Create User`

## Step 2
Enter username: `devops-engineer-01`

Select: **Provide user access to the AWS Management Console**.

## Step 3
Choose **I want to create an IAM user** and set a custom password.

## Step 4
Click **Next**.

On the Permissions page, choose **Add user to group**.

## Step 5
Click **Create Group**.

- Group Name: `DevOpsTeam`
- Attach Policy: `AdministratorAccess` *(for lab only; use least-privilege in production)*

## Step 6
Finish creation.

Download the `.csv` credentials file and save it securely.

## Step 7
Log in with the new IAM user using the console sign-in URL shown.

---

# Task 1.2: Enable MFA for IAM User (AWS Console)

## Step 1
IAM → Users → `devops-engineer-01` → **Security credentials** tab.

## Step 2
Under **Multi-factor authentication (MFA)** select **Assign MFA device**.

## Step 3
Select **Authenticator app** and click **Next**.

## Step 4
Open Google Authenticator or Authy and scan the QR code.

## Step 5
Enter two consecutive 6-digit codes and click **Add MFA**.

## Step 6
Verify by signing out and signing back in.

The MFA prompt should appear.

---

# Task 1.3: Create IAM Role & Attach to EC2 (AWS Console)

## Step 1
IAM → Roles → Create Role

- Trusted entity type: AWS Service
- Use case: EC2

## Step 2
Attach policy:

`AmazonS3ReadOnlyAccess`

Role name:

`EC2-S3-ReadOnly-Role`

## Step 3
Navigate to:

`EC2 → Select Instance → Actions → Security → Modify IAM Role`

## Step 4
Attach:

`EC2-S3-ReadOnly-Role`

Save changes.

## Step 5
SSH into the EC2 instance and run:

```bash
aws s3 ls
```

It should list buckets without configuring credentials.

---

# Task 1.4: Create & Test Custom IAM Policy (AWS CLI)

## Create Policy JSON

```bash
cat > devops-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ec2:Describe*","ec2:StartInstances","ec2:StopInstances"],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject","s3:PutObject","s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-devops-bucket",
        "arn:aws:s3:::my-devops-bucket/*"
      ]
    }
  ]
}
EOF
```

## Create the Policy in AWS

```bash
aws iam create-policy --policy-name DevOpsCustomPolicy --policy-document file://devops-policy.json --region ap-south-1
```

## Attach Policy to User

```bash
aws iam attach-user-policy --user-name devops-engineer-01 --policy-arn arn:aws:iam::ACCOUNT_ID:policy/DevOpsCustomPolicy
```

## Simulate Policy (Test Before Applying)

```bash
aws iam simulate-principal-policy --policy-source-arn arn:aws:iam::ACCOUNT_ID:user/devops-engineer-01 --action-names ec2:StopInstances s3:PutObject
```

---

# 💡 Best Practice Tip

**Least Privilege** is the golden rule.

- Never attach AdministratorAccess in production.
- Create custom policies granting only required permissions.
- Use IAM Access Analyzer to identify unused permissions.
- Rotate access keys every 90 days.
- Never hard-code credentials in source code.
- Use IAM Roles for EC2, ECS, and Lambda.
- Enable CloudTrail to audit all IAM API calls.

---

# 🚀 Pro Tip

Use:

```bash
aws iam get-account-authorization-details
```

to audit the entire account.

Tag IAM resources with:

- Team
- Project
- Environment

for governance, reporting, and cost attribution.
