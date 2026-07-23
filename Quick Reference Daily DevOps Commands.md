# Quick Reference — Daily DevOps Commands

| **Category** | **Command** | **Purpose** |
|--------------|--------------|-------------|
| **Docker** | `docker ps -a` | List all containers |
| | `docker logs -f <name>` | Stream container logs |
| | `docker exec -it <name> bash` | Shell into container |
| **kubectl** | `kubectl get pods -A` | List all pods in all namespaces |
| | `kubectl describe pod <name> -n <ns>` | View pod details & events |
| | `kubectl logs <pod> --previous` | View last crashed container logs |
| | `kubectl rollout undo deploy/<name>` | Rollback deployment to previous revision |
| | `kubectl top nodes` | Show CPU/memory usage of nodes |
| **Terraform** | `terraform plan -out=tfplan` | Preview infrastructure changes |
| | `terraform apply tfplan` | Apply saved plan |
| | `terraform state list` | List managed resources |
| **AWS CLI** | `aws ec2 describe-instances --output table` | List all EC2 instances |
| | `aws logs tail /aws/eks/<cluster>/cluster --follow` | Stream EKS logs |
| | `aws sts get-caller-identity` | Check IAM identity (Who am I?) |
| **Helm** | `helm list -A` | List all Helm releases |
| | `helm rollback <release> <revision> -n <namespace>` | Rollback Helm release |

---

Keep practicing these commands hands-on in your AWS Free Tier account (ap-south-1), document your work in GitHub, and leverage these examples for interviews to showcase your practical skills.

 
