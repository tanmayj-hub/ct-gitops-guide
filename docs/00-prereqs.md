# 00 — Prereqs (Windows + Git Bash)

## Accounts
- AWS account with admin access (for learning)
- GitHub account
- (Optional) Domain you control
  - If you don’t want DNS + HTTPS, you can still finish CI/CD and use the raw ALB DNS name.

## Local tooling (Git Bash)
Install and verify:

- Git + Git Bash
- AWS CLI v2
- Terraform
- kubectl
- Helm
- Kustomize
- jq (optional)
- argocd CLI (optional)

Verify:
```bash
git --version
aws --version
terraform -version
kubectl version --client
helm version
kustomize version
jq --version
argocd version --client
Docker Desktop (required for Stage 6 + Stage 9)

Install Docker Desktop and verify:

docker --version
docker run --rm hello-world
AWS CLI auth

Verify:

aws sts get-caller-identity
Region used

us-east-2

Costs (important)

EKS + NAT Gateway + ALB will cost money if left running.
Follow the destroy runbook in docs/10-destroy.md when done.

Repo names (reference project)

gitops-eks-infra

gitops-eks-apps

ct-gitops-demo-app

---