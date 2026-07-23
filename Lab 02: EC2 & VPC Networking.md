# EC2 & VPC Networking

**Difficulty:** Intermediate  
**Duration:** 3–4 hours  
**Tools:** EC2, VPC, Security Groups, NACLs, NAT Gateway, Application Load Balancer (ALB)

---

## **Objective**

Build a **production-grade 3-tier VPC architecture** featuring:

- Public and private subnets
- NAT Gateway for outbound internet access
- Security Groups and Network Access Control Lists (NACLs)
- Application Load Balancer (ALB)

This setup aligns with best practices in **Infrastructure Management**.

---

## **Tasks Overview**

### **Task 2.1: Create a Custom VPC with Public & Private Subnets**

**Purpose:** Establish an isolated network environment suitable for multi-tier architecture.

#### Steps:

1. **Create VPC**
   - AWS Console → VPC → **Create VPC**
   - Select **VPC and more**
   - Name: `devops-lab-vpc`
   - IPv4 CIDR block: `10.0.0.0/16`

2. **Configure Subnets & Gateways**
   - Set:
     - **Availability Zones:** 2
     - **Public Subnets:** 2
     - **Private Subnets:** 2
     - **NAT Gateways:** 1 per AZ (for high availability)
     - Enable **S3 Gateway endpoint**

3. **Review CIDRs**
   - Confirm auto-assigned CIDRs, for example:
     - Public subnets: `10.0.0.0/20`, `10.0.16.0/20`
     - Private subnets: `10.0.128.0/20`, `10.0.144.0/20`

4. **Create VPC**
   - AWS automatically creates subnets, Internet Gateway (IGW), route tables, and NAT Gateways.

5. **Verify Route Tables**
   - Ensure:
     - Public Route Table has `0.0.0.0/0` routed to IGW
     - Private Route Table has `0.0.0.0/0` routed to NAT Gateway

---

### **Task 2.2: Launch EC2 Instances in Public & Private Subnets**

**Purpose:** Deploy instances to simulate a multi-tier environment.

#### Steps:

1. **Launch Bastion Host (Public Subnet)**
   - EC2 → **Launch Instance**
   - Name: `bastion-host`
   - AMI: Ubuntu 24.04 LTS
   - Instance Type: `t2.micro`
   - Key Pair: Create new → `devops-lab-key`
   - Network Settings:
     - VPC: `devops-lab-vpc`
     - Subnet: `public-subnet-1`
     - Auto-assign Public IP: **Enabled**
   - Security Group: **Create new** → `bastion-sg`
     - Inbound: SSH (port 22) **from your IP only**

2. **Launch Application Server (Private Subnet)**
   - Repeat the process:
     - Name: `app-server`
     - Subnet: `private-subnet-1`
     - No public IP
     - Security Group: **Create new** → `app-sg` (allow SSH from `bastion-sg`)

3. **Connectivity Tests**
   - SSH into the bastion:
     ```bash
     ssh -i devops-lab-key.pem ubuntu@<PUBLIC_IP>
     ```
   - From the bastion, SSH into the private server:
     ```bash
     ssh -i devops-lab-key.pem ubuntu@<PRIVATE_IP>
     ```
   - Confirm private connectivity.

---

### **Task 2.3: Configure Application Load Balancer (ALB)**

**Purpose:** Set up an ALB to distribute traffic to instances.

#### Steps:

1. **Install Web Server on Application Server**
   - Connect to `app-server`:
     ```bash
     sudo apt update && sudo apt install -y nginx
     sudo systemctl start nginx
     ```

2. **Create ALB**
   - EC2 → Load Balancers → **Create Load Balancer**
   - Type: **Application Load Balancer**
   - Name: `devops-alb`
   - Scheme: **Internet-facing**
   - Subnets: Select both **public subnets**

3. **Configure Security Group for ALB**
   - Create new SG: `alb-sg`
   - Inbound Rule: HTTP (port 80) from `0.0.0.0/0`

4. **Configure Target Group**
   - Name: `devops-tg`
   - Target type: **Instances**
   - Protocol: HTTP, Port: 80
   - Register `app-server`

5. **Create ALB**
   - Wait for provisioning (2–3 min)
   - Copy the DNS name and open it in a browser
   - You should see the Nginx default page

---

### **Task 2.4: VPC & Security Configuration via CLI**

**Purpose:** Use AWS CLI for network management and security configurations.

#### Commands:

```bash
# View all VPCs
aws ec2 describe-vpcs --region <region> --output table

# Describe Security Groups
aws ec2 describe-security-groups \
  --filters Name=vpc-id,Values=vpc-XXXXXX \
  --query 'SecurityGroups[*].{Name:GroupName,ID:GroupId}' \
  --output table

# Add inbound rule to Security Group
aws ec2 authorize-security-group-ingress \
  --group-id sg-XXXXXXXX \
  --protocol tcp --port 8080 \
  --cidr 10.0.0.0/16

# List all subnets with CIDR and AZ
aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=vpc-XXXXXX \
  --query 'Subnets[*].{ID:SubnetId,CIDR:CidrBlock,AZ:AvailabilityZone,Public:MapPublicIpOnLaunch}' \
  --output table
```

---

## **Best Practices & Pro Tips**

- **Security Groups** are **stateful**: return traffic is automatically allowed.
- **NACLs** are **stateless**: explicit inbound and outbound rules are required.
- Use Security Groups as primary firewalls; reserve NACLs for subnet-level deny rules (e.g., blocking malicious IPs).
- **Never** open SSH or RDP ports (`0.0.0.0/0`) to the internet; restrict access to specific IPs or use a bastion host.
- In production, place application servers in **private subnets**, exposing only the ALB publicly.

---

This documentation provides a clear, step-by-step guide to building a secure, scalable VPC architecture in AWS. It combines console and CLI instructions for comprehensive learning and operational flexibility.

---
 
