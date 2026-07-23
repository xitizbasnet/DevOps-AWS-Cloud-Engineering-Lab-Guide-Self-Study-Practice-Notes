# DevSecOps — Security Scanning Lab

**Difficulty:** Advanced  
**Duration:** 3–4 hours  
**Tools:** Trivy, OWASP ZAP, SonarQube, IAM Access Analyzer, AWS Config

---

## **Objective**

Integrate security scanning into your CI/CD pipeline for comprehensive security and compliance coverage:

- Container vulnerability scanning with Trivy
- Static Application Security Testing (SAST) with SonarQube
- Secrets scanning
- AWS resource compliance checks with IAM Access Analyzer and AWS Config

---

## **Task 14.1: Container Scanning with Trivy**

### **Installation of Trivy**

```bash
sudo apt-get install wget gnupg -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo 'deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main' | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt-get update && sudo apt-get install -y trivy
```

### **Using Trivy for Container Security Scanning**

- **Scan a Docker image for CVEs with HIGH or CRITICAL severity:**

```bash
trivy image --severity HIGH,CRITICAL nginx:latest
```

- **Scan a local Dockerfile for misconfigurations:**

```bash
trivy config --severity HIGH,CRITICAL .
```

- **Scan filesystem for secrets (e.g., API keys, credentials):**

```bash
trivy fs --scanners secret .
```

- **Generate JSON report for CI/CD integration:**

```bash
trivy image --format json --output trivy-report.json devops-app:v1.0
```

- **Fail the build on CRITICAL vulnerabilities:**

```bash
trivy image --exit-code 1 --severity CRITICAL devops-app:v1.0
echo 'Exit code: '$?
```

- **Scan images in Amazon ECR directly:**

```bash
trivy image --format table <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/devops-app:v1.0
```

---

## **Task 14.2: IAM Access Analyzer & AWS Config**

### **Step-by-step in AWS Console**

1. **Create an IAM Access Analyzer:**

   - Navigate to **IAM > Access Analyzer**
   - Click **Create Analyzer**
   - Set **Type:** Account
   - Name: `devops-analyzer`
   - Click **Create**

2. **Review findings:**

   - Wait for findings to populate
   - Any publicly accessible S3 buckets or IAM roles will appear
   - Review each finding
   - Archive false positives (for intentional public access)
   - Remediate true issues

3. **Set up AWS Config:**

   - Navigate to **AWS Config > Get started**
   - Select **Record all resources**
   - Choose or create an S3 bucket for delivery
   - Click **Next** and enable

4. **Add Config Rules:**

   - Go to **Config > Rules > Add rule**
   - Search for `'iam-password-policy'`
   - Add and evaluate (ensures strong password policies are enforced)

   - Add **'ec2-instance-no-public-ip'** rule to identify EC2 instances with public IPs in private subnets

5. **Review Compliance Dashboard:**

   - Monitor overall compliance status (Compliant vs. Non-Compliant resources)

---

## **Task 14.3: Secrets Scanning in Git (Pre-commit Hooks)**

### **Setup:**

```bash
pip install pre-commit detect-secrets --break-system-packages
```

### **Create Secrets Baseline:**

```bash
detect-secrets scan > .secrets.baseline
```

### **Configure Pre-commit Hooks**

Create `.pre-commit-config.yaml`:

```yaml
repos:
- repo: https://github.com/Yelp/detect-secrets
  rev: v1.4.0
  hooks:
  - id: detect-secrets
    args: ['--baseline', '.secrets.baseline']
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v4.5.0
  hooks:
  - id: trailing-whitespace
  - id: check-merge-conflict
  - id: check-yaml
  - id: check-added-large-files
    args: ['--maxkb=5000']
```

### **Install Git Hooks:**

```bash
pre-commit install
```

### **Run Checks on All Files:**

```bash
pre-commit run --all-files
```

### **Best Practice Tips:**

- **Shift security left**: Detect secrets and security issues early during development.
- **Pre-commit hooks** prevent secrets from reaching repositories.
- **CI pipeline** should include Trivy scans with `--exit-code 1 --severity CRITICAL` to block insecure images.
- Enable **IAM Access Analyzer** in all regions for cross-account exposure detection.
- Use **AWS Config Rules** for continuous compliance monitoring, integrating findings with AWS Security Hub for unified visibility.

---

 
