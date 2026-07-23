# S3, EBS & EFS Storage + ECS Overview

**Difficulty:** Intermediate  
**Duration:** 2–3 hours  
**Tools:** S3, EBS, EFS, ECS Fargate

---

## **Objective**

This lab focuses on working with the three primary AWS storage options:

- **S3** for object storage
- **EBS** for block storage attached to EC2
- **EFS** for shared, NFS-based storage across multiple instances

Additionally, the lab introduces **ECS Fargate** for containerized application deployment, emphasizing serverless container management.

---

## **Tasks Overview**

### **Task 3.1: S3 Bucket — Create, Versioning, Lifecycle**

**Purpose:** Set up an S3 bucket with versioning, lifecycle policies, and encryption.

#### Steps:

1. **Create S3 Bucket**
   - AWS Console → S3 → **Create Bucket**
   - Name: `devops-lab-artifacts-<YOURNAME>` *(must be globally unique)*
   - Region: `ap-south-1`

2. **Configure Access & Versioning**
   - Block all public access (default)
   - Enable **Bucket Versioning**

3. **Upload & Version Files**
   - Upload a file
   - Re-upload the same filename
   - Go to the object, click **Show Versions** to see both versions

4. **Add Lifecycle Rule**
   - Management → Lifecycle rules → **Create rule**
   - Name: `archive-old`
   - Apply to all objects
   - **Lifecycle actions:**
     - Transition to **S3 Standard-Infrequent Access (S3-IA)** after 30 days
     - Transition to **S3 Glacier Flexible** after 90 days
     - Expire objects after 365 days

5. **Enable Server-Side Encryption**
   - Properties → Default Encryption → **SSE-S3 (AES-256)**

---

### **Task 3.2: EBS Volumes — Attach, Mount, Snapshot**

**Purpose:** Provision, attach, and snapshot EBS volumes using AWS CLI.

#### Commands:

```bash
# Create a gp3 EBS volume in the same AZ as your EC2 instance
aws ec2 create-volume \
  --volume-type gp3 \
  --size 10 \
  --availability-zone ap-south-1a \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=devops-data-vol}]'

# Attach the volume to your running EC2 instance
aws ec2 attach-volume \
  --volume-id vol-XXXXXXXXX \
  --instance-id i-XXXXXXXXX \
  --device /dev/xvdf
```

#### On the EC2 Instance:

```bash
# Verify device appears
lsblk

# Format the new volume (only for new volumes)
sudo mkfs -t ext4 /dev/xvdf

# Create mount point
sudo mkdir -p /data

# Mount the volume
sudo mount /dev/xvdf /data

# Verify mount
df -h /data

# Make mount persistent
echo '/dev/xvdf /data ext4 defaults,nofail 0 2' | sudo tee -a /etc/fstab

# Create a snapshot of the volume
aws ec2 create-snapshot \
  --volume-id vol-XXXXXXXXX \
  --description "devops-lab-backup-$(date +%Y%m%d)"
```

---

### **Task 3.3: ECS Fargate — Deploy a Container**

**Purpose:** Deploy a simple Nginx container using ECS Fargate.

#### Steps:

1. **Create ECS Cluster**
   - ECS → Clusters → **Create Cluster**
   - Name: `devops-cluster`
   - Infrastructure: **AWS Fargate**
   - Create

2. **Create a Task Definition**
   - ECS → Task Definitions → **Create new Task Definition**
   - Family: `devops-nginx`
   - Infrastructure: **Fargate**
   - CPU: **0.25 vCPU**
   - Memory: **0.5 GB**
   - Add Container:
     - Name: `nginx`
     - Image URI: `nginx:latest`
     - Port: 80 (TCP)
   - Save Task Definition

3. **Create Service**
   - Cluster → `devops-cluster` → **Services** → **Create Service**
   - Launch type: **Fargate**
   - Task Definition: `devops-nginx`
   - Service name: `nginx-svc`
   - Desired Tasks: `1`

4. **Configure Networking**
   - Select your VPC and **public subnet(s)**
   - Security Group: allow inbound port 80
   - Public IP: **Enabled**
   - Create the service

5. **Access the Web Page**
   - Once the task status is **RUNNING**, select the task
   - Find the **Public IP**
   - Open in a browser — the Nginx welcome page should appear

---

## **Best Practices & Pro Tips**

- **EBS Volumes:** AZ-locked — attach only within the same AZ. For multi-AZ shared storage, use **EFS** (NFS-based, multi-AZ).
- **S3 Buckets:** Enable **versioning** and **MFA-Delete** for critical artifacts. Use **Lifecycle policies** to optimize costs.
- **ECS Fargate:** Eliminates the need to manage EC2 instances. Ideal for microservices and serverless container deployment.
- **Security:** Always restrict access (e.g., SSH/RDP) to specific IPs. Use security groups as primary firewalls.

---

This guide provides a comprehensive walkthrough for managing storage and container deployment on AWS. It balances console and CLI instructions for flexibility and automation.

---

 
