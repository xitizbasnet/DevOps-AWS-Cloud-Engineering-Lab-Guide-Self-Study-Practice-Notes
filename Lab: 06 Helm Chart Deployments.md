# Helm Chart Deployments Lab

**Difficulty:** Advanced  
**Duration:** 2–3 hours  
**Tools:** Helm, Chart Templates, Values, Releases

---

## **Objective**

Learn how to install Helm, create custom Helm charts, manage environment-specific configurations with `values.yaml`, and deploy applications to EKS. This process maps directly to Helm-based deployment responsibilities in modern DevOps workflows.

---

## **Task 6.1: Install Helm & Deploy from Public Chart**

### **Install Helm**

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### **Add Bitnami Repository & Deploy Nginx**

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install my-nginx bitnami/nginx -n devops-apps --set service.type=LoadBalancer --set replicaCount=2
```

### **Check Release Status**

```bash
helm list -n devops-apps
helm status my-nginx -n devops-apps
```

### **Upgrade Release (e.g., Scale to 3 Replicas)**

```bash
helm upgrade my-nginx bitnami/nginx -n devops-apps --set replicaCount=3 --reuse-values
```

### **Rollback to Previous Version**

```bash
helm rollback my-nginx 1 -n devops-apps
helm history my-nginx -n devops-apps
```

---

## **Task 6.2: Create a Custom Helm Chart**

### **Scaffold a New Chart**

```bash
helm create devops-chart
tree devops-chart/
```

### **Edit `values.yaml` for Your Application**

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: alpine
  pullPolicy: IfNotPresent
service:
  type: LoadBalancer
  port: 80
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 250m
    memory: 256Mi
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 5
  targetCPUUtilizationPercentage: 60
```

### **Validate Chart**

```bash
helm lint devops-chart/
helm template devops-chart/ --values devops-chart/values.yaml | head -60
```

### **Install the Custom Chart**

```bash
helm install devops-release ./devops-chart -n devops-apps
```

### **Verify Deployment**

```bash
kubectl get all -n devops-apps
```

---

## **Best Practices & Tips**

- **Lint & Template Validation:** Always run `helm lint` and `helm template` before deploying to production. They help catch syntax errors early.
- **Environment-Specific Values:** Use separate values files like `values-dev.yaml`, `values-staging.yaml`, `values-prod.yaml` for different environments.
- **Version Control:** Store Helm charts in Git repositories and integrate with CI/CD pipelines (e.g., ArgoCD, GitHub Actions).
- **Idempotent Deployments:** Use `helm upgrade --install` to ensure deployments are repeatable and safe in automation pipelines.

---

This guide provides a comprehensive overview of managing Helm charts for scalable, maintainable deployments on Kubernetes, emphasizing best practices for production readiness.

---

 
