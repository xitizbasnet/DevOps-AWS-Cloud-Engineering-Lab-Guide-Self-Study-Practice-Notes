# Bash & Python Automation Scripting Lab

**Difficulty:** Intermediate  
**Duration:** 2–3 hours  
**Tools:** Bash, Python, boto3, cron, AWS CLI

---

## **Objective**

Create production-grade Bash and Python scripts to automate AWS operations such as EC2 management, S3 cleanup, EKS health checks, and scheduled tasks.  
Maps to: **Automation & Tooling** in the JD.

---

## **Task 13.1: Bash — EC2 Health Check Script**

### **Script: `ec2-health-check.sh`**

```bash
#!/usr/bin/env bash
# ec2-health-check.sh — List stopped EC2s and optionally start them
set -euo pipefail

REGION='ap-south-1'
TAG_KEY='Env'
TAG_VAL='prod'

echo "========= EC2 Health Report: $(date) ========="

# Get stopped instances in prod
STOPPED=$(aws ec2 describe-instances \
  --region "$REGION" \
  --filters "Name=tag:$TAG_KEY,Values=$TAG_VAL" \
           "Name=instance-state-name,Values=stopped" \
  --query 'Reservations[*].Instances[*].[InstanceId,Tags[?Key==`Name`].Value|[0]]' \
  --output text)

if [[ -z "$STOPPED" ]]; then
  echo 'All prod instances are running.'
else
  echo 'STOPPED INSTANCES:'
  echo "$STOPPED"
  read -r -p 'Start all stopped instances? (y/N): ' CONFIRM
  if [[ "$CONFIRM" == 'y' ]]; then
    echo "$STOPPED" | awk '{print $1}' | xargs -r -n 1 aws ec2 start-instances --region "$REGION" --instance-ids
    echo 'Start command sent.'
  fi
fi

# Schedule with cron (run every 15 min)
# */15 * * * * /home/ubuntu/scripts/ec2-health-check.sh >> /var/log/ec2-health.log 2>&1
```

### **Notes:**
- Uses `set -euo pipefail` for robust error handling.
- Fetches stopped EC2 instances with specific tags.
- Prompts to start stopped instances.
- Can be scheduled with cron to run periodically.

---

## **Task 13.2: Python — S3 Cost Cleanup with boto3**

### **Script: `s3_cleanup.py`**

```python
#!/usr/bin/env python3
# s3_cleanup.py — Delete objects older than N days from a bucket

import boto3
from datetime import datetime, timezone, timedelta

# Configuration
BUCKET = 'devops-lab-artifacts-yourname'
MAX_AGE_DAYS = 90
DRY_RUN = True  # Set to False to actually delete

s3 = boto3.client('s3', region_name='ap-south-1')
cutoff = datetime.now(timezone.utc) - timedelta(days=MAX_AGE_DAYS)

paginator = s3.get_paginator('list_objects_v2')
delete_keys = []

for page in paginator.paginate(Bucket=BUCKET):
    for obj in page.get('Contents', []):
        if obj['LastModified'] < cutoff:
            delete_keys.append({'Key': obj['Key']})
            print(f'OLD: {obj["Key"]} ({obj["LastModified"].date()})')

print(f'\nTotal objects to delete: {len(delete_keys)}')

if not DRY_RUN and delete_keys:
    # Batch delete (max 1000 objects per call)
    for i in range(0, len(delete_keys), 1000):
        batch = delete_keys[i:i+1000]
        s3.delete_objects(Bucket=BUCKET, Delete={'Objects': batch})
        print(f'Deleted batch {i//1000 + 1}')
    print('Cleanup complete.')
else:
    print('[DRY RUN] No objects deleted. Set DRY_RUN=False to delete.')
```

### **Notes:**
- Uses `set -euo pipefail` best practices in Bash scripts.
- Implements paginators in boto3 for efficient listing of S3 objects.
- Supports dry-run mode to prevent accidental deletions.
- Designed to run periodically via cron or AWS EventBridge.

---

## **Best Practices & Tips**

- **Error Handling:** Always use `set -euo pipefail` in Bash scripts for safety.
- **Dry Run Mode:** Use a `DRY_RUN` flag in Python scripts to test scripts before executing destructive actions.
- **AWS Paginators:** Always use paginators in boto3 for list operations exceeding 1000 results.
- **Scheduling:** Automate scripts with cron or AWS EventBridge for serverless execution via Lambda or EC2.

---

 
