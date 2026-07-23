# Jenkins CI/CD Pipeline Lab

**Difficulty:** Advanced  
**Duration:** 3–4 hours  
**Tools:** Jenkins, Jenkinsfile, Blue Ocean, Webhooks, Agents

---

## **Objective**

- Install Jenkins on an EC2 instance
- Configure a multibranch pipeline using a Jenkinsfile
- Set up GitHub webhooks for automatic trigger
- Deploy applications to EKS
- Fulfills Jenkins and pipeline automation requirements in the JD

---

## **Task 9.1: Install Jenkins on EC2**

### **Step-by-Step Installation on Ubuntu EC2 (Recommended: t3.medium)**

```bash
# Update package list and install OpenJDK 17
sudo apt update && sudo apt install -y openjdk-17-jdk

# Add Jenkins repo key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins repository
echo 'deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/' | sudo tee /etc/apt/sources.list.d/jenkins.list

# Install Jenkins
sudo apt update && sudo apt install -y jenkins

# Enable and start Jenkins service
sudo systemctl enable --now jenkins

# Retrieve initial admin password for first login
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### **Access Jenkins**

- Open a browser and navigate to: `http://<EC2_PUBLIC_IP>:8080`
- Use the initial admin password retrieved above
- Install suggested plugins and the **Blue Ocean** plugin for a visual pipeline interface
- Add Docker to Jenkins for container build capabilities:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

## **Task 9.2: Write Declarative Jenkinsfile**

Create a `Jenkinsfile` in your repository:

```groovy
pipeline {
    agent any
    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = 'devops-app'
        EKS_CLUSTER = 'devops-eks'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Source code checked out'
            }
        }
        stage('Test') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'python -m pytest tests/ -v'
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                withAWS(credentials: 'aws-creds', region: AWS_REGION) {
                    sh '''
                    aws ecr get-login-password | docker login --username AWS --password-stdin \
                    $(aws sts get-caller-identity --query Account --output text).dkr.ecr.$AWS_REGION.amazonaws.com
                    docker build -t $ECR_REPO:$IMAGE_TAG .
                    docker push $ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }
        stage('Deploy to EKS') {
            when { branch 'main' }
            steps {
                withAWS(credentials: 'aws-creds', region: AWS_REGION) {
                    sh '''
                    aws eks update-kubeconfig --name $EKS_CLUSTER --region $AWS_REGION
                    kubectl set image deployment/devops-app app=$ECR_REPO:$IMAGE_TAG -n devops-apps
                    kubectl rollout status deployment/devops-app -n devops-apps
                    '''
                }
            }
        }
    }
    post {
        failure {
            sh 'kubectl rollout undo deployment/devops-app -n devops-apps || true'
            echo 'Pipeline FAILED — rollback triggered'
        }
        always {
            cleanWs()
        }
    }
}
```

### **Additional Tips**

- **Credentials:** Store AWS credentials securely in Jenkins Credentials (not hardcoded in the Jenkinsfile)
- **Webhooks:** Configure GitHub webhooks under **Manage Jenkins > Configure System > GitHub** to trigger pipelines automatically on push
- **UI:** Use **Blue Ocean** for a visual pipeline view
- **Backup:** Regularly backup Jenkins home directory (`/var/lib/jenkins`) — including credentials and job configs

---

This setup allows automatic, secure, and reliable CI/CD for deploying containerized applications to AWS EKS with Jenkins.

 
