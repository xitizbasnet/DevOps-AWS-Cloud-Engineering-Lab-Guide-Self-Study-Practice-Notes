# Terraform Infrastructure as Code (IaC) Lab

**Difficulty:** Advanced  
**Duration:** 3–4 hours  
**Tools:** Terraform, tfstate, Modules, Variables, Workspaces

---

## **Objective**

Provision AWS infrastructure—including VPC, EC2, and S3—using Terraform. Utilize modules, remote state with S3 backend, and workspaces for environment separation. This aligns with IaC best practices and meets the requirements of modern cloud deployment.

---

## **Task 7.1: Install Terraform & Configure AWS Provider**

### **Install Terraform**

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main' | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y terraform
terraform version
```

### **Set Up Project Structure**

```bash
mkdir -p ~/tf-lab/{modules/ec2,envs/dev}
cd ~/tf-lab
```

### **Create `main.tf` with Provider & Backend Configuration**

```hcl
terraform {
  required_version = ">= 1.6"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  backend "s3" {
    bucket = "devops-tf-state-<YOURNAME>"
    key    = "dev/terraform.tfstate"
    region = "ap-south-1"
    encrypt = true
  }
}

provider "aws" {
  region = var.aws_region
}
```

---

## **Task 7.2: Create EC2 + VPC with Terraform**

### **Define Variables in `variables.tf`**

```hcl
variable "aws_region" { default = "ap-south-1" }
variable "env" { default = "dev" }
variable "instance_type" { default = "t2.micro" }
```

### **Create Infrastructure in `ec2.tf`**

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners = ["099720109477"] # Canonical (Ubuntu)
  filter {
    name = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  enable_dns_hostnames = true
  tags = { Name = "tf-${var.env}-vpc" }
}

resource "aws_subnet" "public" {
  vpc_id = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = true
  tags = { Name = "tf-${var.env}-public-subnet" }
}

resource "aws_instance" "app" {
  ami = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id = aws_subnet.public.id
  tags = { Name = "tf-${var.env}-app-server", Env = var.env }
}
```

---

## **Terraform Workflow**

### **Initialize, Format, Validate, and Plan**

```bash
terraform init
terraform fmt        # Auto-format configuration files
terraform validate   # Check syntax and correctness
terraform plan -out=tfplan
```

### **Apply Infrastructure**

```bash
terraform apply tfplan
```

### **Use Workspaces for Environment Separation**

```bash
terraform workspace new staging
terraform workspace list
terraform apply -var='env=staging'
```

### **Clean Up Resources**

```bash
terraform destroy -auto-approve
```

---

## **Best Practice Tips**

- **Remote State Management:** Store Terraform state in S3 with DynamoDB locking to prevent concurrent modifications.
- **Plan & Apply:** Always run `terraform plan` and review the plan before executing `terraform apply`. Save plans for CI/CD pipelines.
- **Formatting & Validation:** Use `terraform fmt` and `terraform validate` as pre-commit hooks to enforce code quality.
- **Resource Tagging:** Tag resources with `Env`, `Team`, and `ManagedBy=Terraform` for better manageability and auditing.
- **Environment Separation:** Use workspaces or separate state files for dev, staging, and production environments. Never share state across environments.

---

This structured guide ensures best practices and clear steps for provisioning AWS infrastructure via Terraform, suitable for advanced learners and automation-focused teams.

 
