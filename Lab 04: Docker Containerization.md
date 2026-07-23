# Docker Containerization Lab

**Difficulty:** Intermediate  
**Duration:** 2–3 hours  
**Tools:** Docker, Dockerfile, Docker Compose, AWS ECR

---

## **Objective**

Learn how to build, tag, push, and run Docker containers. Develop multi-stage Dockerfiles, utilize Docker Compose for local development, and push container images to AWS Elastic Container Registry (ECR). This covers core containerization and orchestration concepts.

---

## **Task 4.1: Install Docker & Run First Container**

### **Install Docker on Ubuntu EC2**

```bash
sudo apt update && sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo usermod -aG docker $USER && newgrp docker
```

### **Verify Installation**

```bash
docker --version
docker run hello-world
```

### **Explore Container Lifecycle**

```bash
docker pull nginx:alpine
docker run -d --name my-nginx -p 8080:80 nginx:alpine
docker ps
docker logs my-nginx
docker exec -it my-nginx sh
docker stop my-nginx && docker rm my-nginx
```

---

## **Task 4.2: Write Multi-Stage Dockerfile**

### **Create Sample Flask Application**

```bash
mkdir -p ~/devops-app && cd ~/devops-app
cat > app.py << 'EOF'
from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello():
    return 'DevOps Lab App v1.0'
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

cat > requirements.txt << 'EOF'
flask==3.0.0
gunicorn==21.2.0
EOF
```

### **Dockerfile with Multi-Stage Build**

```dockerfile
# Stage 1 — Build dependencies
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2 — Production image
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY app.py .
ENV PATH=/root/.local/bin:$PATH
EXPOSE 5000
USER nobody
CMD ["gunicorn", "-w", "2", "-b", "0.0.0.0:5000", "app:app"]
```

### **Build & Run Docker Image**

```bash
docker build -t devops-app:v1.0 .
docker images devops-app
docker run -d -p 5000:5000 --name devops-app devops-app:v1.0
curl http://localhost:5000
```

---

## **Task 4.3: Push Image to AWS ECR**

### **Create ECR Repository**

```bash
aws ecr create-repository \
  --repository-name devops-app \
  --region ap-south-1 \
  --image-scanning-configuration scanOnPush=true
```

### **Authenticate Docker to ECR**

```bash
aws ecr get-login-password --region ap-south-1 | \
docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com
```

*(Replace `<ACCOUNT_ID>` with your AWS account ID)*

### **Tag & Push Image**

```bash
docker tag devops-app:v1.0 <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/devops-app:v1.0
docker push <ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/devops-app:v1.0
```

### **Verify Scan Results**

```bash
aws ecr describe-image-scan-findings \
  --repository-name devops-app \
  --image-id imageTag=v1.0 \
  --region ap-south-1
```

---

## **Best Practices & Tips**

- **Multi-Stage Builds:** Keep production images small and secure by using multi-stage Docker builds. Fewer layers reduce vulnerabilities.
- **Run Containers as Non-Root:** Use `USER nobody` or a dedicated user for better security.
- **Health Checks:** Add `HEALTHCHECK` in your Dockerfile to enable automatic container health monitoring.
- **Image Scanning:** Enable **scanOnPush** in ECR to identify CVEs before deployment.
- **.dockerignore:** Use this file to exclude unnecessary files like `.git`, `node_modules`, and secrets from the build context for security and efficiency.

---

This comprehensive guide streamlines the containerization process from installation to deployment on AWS, emphasizing best practices for secure and efficient Docker usage.

---

 
