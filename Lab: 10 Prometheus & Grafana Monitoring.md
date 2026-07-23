# Prometheus & Grafana Monitoring Lab

**Difficulty:** Advanced  
**Duration:** 3–4 hours  
**Tools:** Prometheus, Grafana, AlertManager, kube-state-metrics

---

## **Objective**

- Deploy the Prometheus Monitoring Stack on EKS using Helm
- Build Grafana dashboards for visualization
- Configure AlertManager for PagerDuty/email alerts
- Fulfill the Observability & Monitoring requirements in the JD

---

## **Task 10.1: Deploy kube-prometheus-stack via Helm**

### **Step-by-step Commands**

```bash
# Add Prometheus Helm chart repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Create monitoring namespace
kubectl create namespace monitoring

# Install the kube-prometheus-stack with custom settings
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --set grafana.adminPassword=DevOps@2024 \
  --set prometheus.prometheusSpec.retention=30d \
  --set alertmanager.enabled=true

# Verify pods are running
kubectl get pods -n monitoring

# List services to confirm exposure
kubectl get svc -n monitoring

# Port-forward Grafana locally
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring &

# Access Grafana at: http://localhost:3000 (user: admin, password: DevOps@2024)

# Port-forward Prometheus locally
kubectl port-forward svc/kube-prometheus-stack-prometheus 9090:9090 -n monitoring &

# Access Prometheus at: http://localhost:9090
```

---

## **Task 10.2: Key PromQL Queries for DevOps**

Run these queries in the Prometheus Web UI (`http://localhost:9090`) to monitor system health:

```promql
# CPU usage per pod
sum(rate(container_cpu_usage_seconds_total{namespace='devops-apps'}[5m])) by (pod)

# Memory usage per pod (bytes)
sum(container_memory_working_set_bytes{namespace='devops-apps'}) by (pod)

# Node CPU utilization (%)
100 - (avg by(node) (rate(node_cpu_seconds_total{mode='idle'}[5m])) * 100)

# Pod restarts in the last hour
increase(kube_pod_container_status_restarts_total{namespace='devops-apps'}[1h])

# HTTP 5xx error rate
sum(rate(http_requests_total{status=~'5..'}[5m])) / sum(rate(http_requests_total[5m]))

# Deployment replicas available vs desired
kube_deployment_status_replicas_available / kube_deployment_spec_replicas
```

---

## **Task 10.3: Configure AlertManager Rules**

Create alert rules for critical conditions:

```yaml
cat > alert-rules.yaml << 'EOF'
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: devops-alerts
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  groups:
  - name: devops.rules
    rules:
    - alert: HighCPUUsage
      expr: sum(rate(container_cpu_usage_seconds_total[5m])) by (pod) > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: High CPU on {{ $labels.pod }}
        description: CPU > 80% for 5 min on {{ $labels.pod }}
    - alert: PodCrashLooping
      expr: increase(kube_pod_container_status_restarts_total[1h]) > 5
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: Pod {{ $labels.pod }} is crash-looping
EOF
kubectl apply -f alert-rules.yaml
```

---

## **Best Practices & Tips**

- **Grafana Dashboard:** Import Dashboard ID **15760** (Kubernetes All-in-One) immediately after setup for comprehensive metrics covering CPU, memory, network, and pod health.
- **Monitoring Signals:** Focus on the four Golden Signals: **Latency, Traffic, Errors, Saturation (LTES)**.
- **Alerting:** Always set a `'for'` duration in alert rules (e.g., 5m, 2m) to prevent false positives from brief spikes.
- **Notifications:** Use Grafana alerting to notify via Slack, PagerDuty, or email.
- **Long-term Storage:** Store Prometheus data for over 30 days using Thanos or AWS Managed Prometheus for extended observability.

---

 
