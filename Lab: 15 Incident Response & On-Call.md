# Incident Response & On-Call Lab

**Difficulty:** Advanced  
**Duration:** 2–3 hours  
**Tools:** Runbooks, Post-Mortems, PagerDuty, CloudWatch, kubectl

---

## **Objective**

Practice a structured incident response process: **detect**, **triage**, **remediate**, and **write a post-mortem**. Simulate a production outage (Pod CrashLoop) and follow a runbook to resolve it, fulfilling Incident Response, Post-mortems, and On-Call requirements.

---

## **Task 15.1: Incident Response Runbook — Pod CrashLoop**

### **Scenario:**  
Alert fires: `PodCrashLooping` — `devops-app` pod restarting continuously.

---

### **Step-by-step Incident Response**

#### **STEP 1 — Triage: Identify the affected pod**

```bash
kubectl get pods -n devops-apps
```

- Look for pods with `STATUS = CrashLoopBackOff`.

---

#### **STEP 2 — Get current logs (including from previous crash)**

```bash
kubectl logs <POD_NAME> -n devops-apps --previous
```

- Review logs for errors like Out of Memory (OOMKilled), missing ConfigMaps, or image pull issues.

---

#### **STEP 3 — Describe the pod for events**

```bash
kubectl describe pod <POD_NAME> -n devops-apps
```

- Look for events indicating issues:
  - `OOMKilled` (memory overuse)
  - `ImagePullBackOff` (image fetch errors)
  - Missing ConfigMap or Secret references

---

#### **STEP 4 — Check resource limits vs actual usage**

```bash
kubectl top pod <POD_NAME> -n devops-apps
```

- Verify if resource limits are appropriate or need adjustment.

---

#### **STEP 5 — Check recent events at namespace level**

```bash
kubectl get events -n devops-apps --sort-by='.lastTimestamp' | tail -20
```

- Spot any other underlying issues or warnings.

---

#### **STEP 6 — Common Fixes**

**A. Memory limit too low (OOMKilled):**

```bash
kubectl patch deployment devops-app -n devops-apps -p \
'{"spec":{"template":{"spec":{"containers":[{"name":"app","resources":{"limits":{"memory":"512Mi"}}}]}}}}'
```

**B. Wrong image tag (rollback):**

```bash
kubectl rollout undo deployment/devops-app -n devops-apps
kubectl rollout status deployment/devops-app -n devops-apps
```

**C. Missing Secret (create secret):**

```bash
kubectl get secrets -n devops-apps
kubectl create secret generic db-creds \
  --from-literal=DB_HOST=mydb.ap-south-1.rds.amazonaws.com \
  --from-literal=DB_PASSWORD=secret123 \
  -n devops-apps
```

---

#### **STEP 7 — Verify recovery**

```bash
kubectl get pods -n devops-apps -w
kubectl rollout status deployment/devops-app -n devops-apps
```

- Pod should recover and become healthy.

---

## **Task 15.2: Post-Mortem Template**

### **Incident Title:**  
`PodCrashLoop — devops-app — Production`

### **Date & Duration:**  
`2024-06-15 14:32 IST` — `15:05 IST` (33 min downtime)

### **Severity:**  
`P1 — Customer-facing API unavailable`

### **Detected By:**  
`PodCrashLooping alert fired via AlertManager → PagerDuty`

### **Root Cause:**  
Memory limit (128Mi) too low for spike in traffic; OOMKilled triggered crash loop.

### **Timeline:**

| Time        | Event                                        |
|-------------|----------------------------------------------|
| 14:32       | Alert fires                                  |
| 14:38       | Engineer paged                              |
| 14:45       | Triage initiated                            |
| 14:58       | Fix applied (increase memory limit)         |
| 15:05       | Issue resolved, pod stabilized              |

### **Impact:**  
API returning 503 errors for approximately 33 minutes, affecting ~1,200 requests. No data loss.

### **Resolution:**  
Memory limit increased to 512Mi. Rollout triggered, pod stabilized.

### **Action Items:**

1. Tune resource limits based on load testing data.
2. Add Horizontal Pod Autoscaler (HPA) based on memory usage.
3. Add runbook link to alert for quick reference.

---

## **Best Practices & Pro Tips**

- Follow the **DETCT → TRIAGE → CONTAIN → RESOLVE → POST-MORTEM** cycle.
- **Communicate early**: Share incident status within first 5 minutes.
- Always run `kubectl logs --previous` for CrashLoopBackOff pods.
- Write blameless post-mortems focusing on systemic improvements, not individuals.
- Document every incident with timeline, root cause, and corrective actions in shared documentation (Confluence, Notion).

---

## **On-Call Rotation & Incident Management Tips**

- Set up escalation paths in **PagerDuty** (L1 → L2 → Manager).
- Link every alert to a runbook.
- Reduce alert fatigue by tuning thresholds or fixing root causes.
- Track **MTTD (Mean Time to Detect)** and **MTTR (Mean Time to Recover)** as KPIs for response health.

---

 
