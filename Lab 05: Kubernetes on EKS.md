# Kubernetes on EKS Lab

**Difficulty:** Advanced  
**Duration:** 4–5 hours  
**Tools:** EKS, kubectl, eksctl, Deployments, HPA, Services

---

## **Objective**

Provision an Amazon EKS cluster, deploy applications using Kubernetes Deployments and Services, configure Horizontal Pod Autoscaling (HPA), and manage configurations with ConfigMaps and Secrets. This covers core concepts of Containerization & Orchestration, multi-cloud deployment, and scalable architecture.

---

## **Task 5.1: Create EKS Cluster with eksctl**

### **Install eksctl**

```bash
curl --silent --location 'https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz' | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### **Install kubectl**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

### **Create EKS Cluster (Approx. 15 mins)**

```bash
eksctl create cluster \
  --name devops-eks \
  --region ap-south-1 \
  --version 1.29 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2 --nodes-min 1 --nodes-max 3 \
  --managed
```

### **Configure kubectl**

```bash
aws eks update-kubeconfig --name devops-eks --region ap-south-1
kubectl get nodes
# Should display 2 nodes in Ready status
```

---

## **Task 5.2: Deploy Application with Deployment & Service**

### **Create Namespace**

```bash
kubectl create namespace devops-apps
```

### **Deployment Manifest (`deployment.yaml`)**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-app
  namespace: devops-apps
  labels:
    app: devops-app
    version: v1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devops-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: devops-app
    spec:
      containers:
        - name: app
          image: nginx:alpine
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 250m
              memory: 256Mi
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 20
```

### **Apply Deployment**

```bash
kubectl apply -f deployment.yaml
```

### **Expose Deployment via LoadBalancer Service**

```bash
kubectl expose deployment devops-app -n devops-apps --type=LoadBalancer --port=80 --name=devops-app-svc
```

### **Monitor Rollout & Get External URL**

```bash
kubectl rollout status deployment/devops-app -n devops-apps
kubectl get svc -n devops-apps
# Copy the EXTERNAL-IP for access
```

---

## **Task 5.3: Horizontal Pod Autoscaler (HPA) Setup**

### **Install Metrics Server**

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl get deployment metrics-server -n kube-system
```

### **Create HPA (Scale when CPU > 50%)**

```bash
kubectl autoscale deployment devops-app -n devops-apps --cpu-percent=50 --min=2 --max=6
kubectl get hpa -n devops-apps
```

### **Useful `kubectl` Commands**

```bash
kubectl get pods -n devops-apps -o wide
kubectl describe pod <POD_NAME> -n devops-apps
kubectl logs <POD_NAME> -n devops-apps --tail=50 -f
kubectl top pods -n devops-apps
kubectl top nodes
```

---

## **Best Practice Tips**

- **Resource Requests & Limits:** Always specify `resources.requests` and `resources.limits` for containers to prevent resource starvation.
- **RollingUpdate Strategy:** Use `maxUnavailable: 0` for zero-downtime updates during deployments.
- **Probes:** Configure both `readinessProbe` and `livenessProbe` to ensure only healthy pods receive traffic and to automatically restart unhealthy pods.
- **Namespaces:** Use dedicated namespaces for environment isolation, avoiding the default namespace in production.
- **Cluster Maintenance:** EKS manages node patching, reducing operational overhead.

---

This guide provides a comprehensive walkthrough from cluster creation to deploying scalable, resilient applications on EKS, aligning with best practices for production-grade Kubernetes deployments.

---

 
