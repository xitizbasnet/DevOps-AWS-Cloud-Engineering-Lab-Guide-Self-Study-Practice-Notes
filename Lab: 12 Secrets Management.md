# Secrets Management Lab

**Difficulty:** Advanced  
**Duration:** 2–3 hours  
**Tools:** AWS Secrets Manager, Parameter Store, Vault, External Secrets Operator (ESO)

---

## **Objective**

- Store, rotate, and retrieve secrets using AWS Secrets Manager
- Manage parameters with SSM Parameter Store
- Integrate secrets into Kubernetes clusters via External Secrets Operator (ESO)
- Map to Secrets Management and DevSecOps practices

---

## **Task 12.1: AWS Secrets Manager (Console + CLI)**

### **Step-by-step Instructions**

1. **Store a new secret in Secrets Manager**
   - Navigate to **Secrets Manager > Store a new secret**
   - Choose **Secret type:** Other
   - Enter **Key/Value pairs:**
     - `DB_HOST = mydb.ap-south-1.rds.amazonaws.com`
     - `DB_PASSWORD = SuperSecure#2024`
   - Click **Next**

2. **Configure Secret Name & Rotation**
   - Set **Secret name:** `/devops/prod/db-credentials`
   - Leave **Automatic rotation** as **off** (requires Lambda for rotation)
   - Click **Next**

3. **Review & Create**
   - Confirm details
   - Click **Store**
   
4. **Note the ARN** (Amazon Resource Name) of the created secret for future reference

5. **Retrieve the secret via CLI**
   ```bash
   aws secretsmanager get-secret-value --secret-id /devops/prod/db-credentials
   ```

6. **Application Best Practice**
   - Never hardcode secrets in code
   - Always call Secrets Manager API at runtime to fetch credentials

---

## **Task 12.2: SSM Parameter Store & Kubernetes Integration**

### **Store a SecureString Parameter**

```bash
aws ssm put-parameter \
  --name '/devops/prod/api-key' \
  --value 'sk-prod-abc123xyz456' \
  --type SecureString \
  --key-id alias/aws/ssm \
  --region ap-south-1
```

### **Retrieve the Parameter (Decrypted)**

```bash
aws ssm get-parameter \
  --name '/devops/prod/api-key' \
  --with-decryption \
  --query 'Parameter.Value' \
  --output text
```

### **Retrieve All Params Under a Path**

```bash
aws ssm get-parameters-by-path \
  --path '/devops/prod/' \
  --with-decryption \
  --recursive
```

### **Integrate with Kubernetes using External Secrets Operator**

Create an ExternalSecret manifest:

```yaml
cat > external-secret.yaml << 'EOF'
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
  namespace: devops-apps
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: db-credentials-k8s
    creationPolicy: Owner
  data:
    - secretKey: DB_HOST
      remoteRef:
        key: /devops/prod/db-credentials
        property: DB_HOST
    - secretKey: DB_PASSWORD
      remoteRef:
        key: /devops/prod/db-credentials
        property: DB_PASSWORD
EOF
```

Apply the ExternalSecret manifest:

```bash
kubectl apply -f external-secret.yaml
```

---

## **Best Practices & Tips**

- **Never store secrets in environment variables baked into Docker images or in Git repositories.**
- Use **AWS Secrets Manager** for sensitive data like database passwords, API keys, and TLS certificates.
- Use **SSM Parameter Store** for non-sensitive configurations such as hostnames and feature flags. Use `SecureString` for sensitive values.
- In Kubernetes, **never create secrets manually**; use **External Secrets Operator** to automatically sync secrets from AWS Secrets Manager, ensuring secrets stay in sync and reducing manual overhead.

---

 
