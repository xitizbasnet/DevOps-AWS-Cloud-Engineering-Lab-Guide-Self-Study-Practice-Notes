# AWS CloudWatch Observability Lab

**Difficulty:** Intermediate  
**Duration:** 2–3 hours  
**Tools:** CloudWatch, Alarms, Dashboards, Logs Insights, Container Insights

---

## **Objective**

- Set up CloudWatch alarms for EC2 instances
- Create custom dashboards
- Run Log Insights queries
- Enable Container Insights for EKS

---

## **Task 11.1: Create CloudWatch Alarm (AWS Console)**

### **Steps to Create an EC2 CPU Utilization Alarm**

1. **Navigate to CloudWatch**
   - Go to **CloudWatch > Alarms > Create Alarm**

2. **Select Metric**
   - Choose **EC2 > Per-Instance Metrics**
   - Select **CPUUtilization** for your EC2 instance

3. **Configure Alarm**
   - Select your specific EC2 instance
   - Set **Period**: 5 minutes
   - Set **Statistic**: Average
   - Define Condition:
     - Threshold: **Greater/Equal than 80**
   - Action:
     - Create a **new SNS topic** named `devops-alerts`
     - Enter your email address for notifications

4. **Confirm Subscription**
   - Check your email and confirm the SNS subscription

5. **Name and Create Alarm**
   - Name your alarm: **HighCPU-EC2**
   - Click **Create**

6. **Test the Alarm**
   - Stress the EC2 with:
     ```bash
     sudo apt install -y stress && stress --cpu 2 --timeout 60
     ```
   - Watch the alarm state change from **OK** to **ALARM**
   - Confirm receipt of email notification

---

## **Task 11.2: CloudWatch Logs Insights Queries (CLI)**

### **Run Logs Insights Query**

```bash
aws logs start-query \
  --log-group-name /var/log/nginx/access.log \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 50'
```

### **Retrieve Query Results**

Replace `<QUERY_ID>` with the ID returned from the previous command:

```bash
aws logs get-query-results --query-id <QUERY_ID>
```

---

## **Enable Container Insights on EKS**

```bash
ClusterName=devops-eks
RegionName=ap-south-1

aws eks create-addon \
  --cluster-name $ClusterName \
  --addon-name amazon-cloudwatch-observability \
  --region $RegionName
```

---

## **Push Custom Metrics to CloudWatch**

```bash
aws cloudwatch put-metric-data \
  --metric-name ActiveUsers \
  --namespace DevOpsApp \
  --value 142 \
  --dimensions Environment=prod \
  --region ap-south-1
```

---

## **Best Practices & Tips**

- **Key EC2 Metrics:** Always set alarms on **CPUUtilization**, **NetworkIn/Out**, **DiskReadOps**, and **StatusCheckFailed**.
- **Composite Alarms:** Use **composite alarms** to reduce alert noise, e.g., trigger PagerDuty only when **both** CPU **and** Memory are high.
- **Logs Insights:** Master the syntax for faster querying and troubleshooting.
- **Container Insights:** Enable pod-level metrics via Container Insights for quick, AWS-native observability without deploying Prometheus.

---

 
