# CI/CD with GitHub Actions Lab

**Difficulty:** Intermediate  
**Duration:** 2–3 hours  
**Tools:** GitHub Actions, Workflows, OIDC, ECR, EKS

---

## **Objective**

Build a complete CI/CD pipeline using GitHub Actions that:
- Tests your application
- Builds a Docker image
- Pushes the image to Amazon ECR
- Deploys to Amazon EKS
- Implements automated testing and rollback strategies

---

## **Task 8.1: Create GitHub Actions Workflow**

### **Step 1: Create Workflow File**

```bash
mkdir -p .github/workflows
cat > .github/workflows/ci-cd.yml << 'EOF'
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: devops-app
  EKS_CLUSTER: devops-eks
  K8S_NAMESPACE: devops-apps
jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install -r requirements.txt
      - run: python -m pytest tests/ -v --tb=short

  build-and-push:
    name: Build & Push to ECR
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}
      - name: Login to ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      - name: Build, tag, and push image
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

  deploy:
    name: Deploy to EKS
    needs: build-and-push
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}
      - name: Update kubeconfig
        run: aws eks update-kubeconfig --name $EKS_CLUSTER --region $AWS_REGION
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/devops-app app=${{ needs.build-and-push.outputs.image }} -n $K8S_NAMESPACE
          kubectl rollout status deployment/devops-app -n $K8S_NAMESPACE --timeout=5m
      - name: Rollback on failure
        if: failure()
        run: kubectl rollout undo deployment/devops-app -n $K8S_NAMESPACE
EOF
```

### **Step 2: Commit & Push**

```bash
git add . && git commit -m 'feat: add CI/CD pipeline with GitHub Actions'
git push origin main
```

---

## **Best Practice Tips**

- **Use OIDC Authentication:** Leverage GitHub OIDC to securely assume AWS roles without long-lived credentials.
- **Workflow Structuring:** Separate jobs for testing, building, and deploying. Use `needs:` to enforce proper order.
- **Automate Rollbacks:** Always implement `kubectl rollout undo` to revert deployments on failures.
- **Branch Protection:** Enforce CI passing before merging into `main`.
- **Caching Dependencies:** Use `actions/cache` to speed up pip/npm installs and reduce build times by 60–70%.

---

This setup ensures a secure, automated, and reliable CI/CD pipeline integrating GitHub Actions with AWS services for containerized applications.

 
