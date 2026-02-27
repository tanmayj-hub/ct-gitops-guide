# 03 — Infra stack (VPC + EKS + ECR + Route 53 + ACM + Secrets Manager + GitHub OIDC role)

This stage applies:
- `gitops-eks-infra/envs/dev`

It provisions:
- VPC (2 AZs, public/private subnets, 1 NAT Gateway)
- EKS cluster (Kubernetes 1.34) + managed node group (AL2023)
- IRSA enabled (OIDC provider for pods)
- ECR repo: `ct-gitops/demo-app`
- Route 53 hosted zone for `root_domain` (defaults to `p1.cloudwithtanmay.com`)
- ACM wildcard cert for `*.${root_domain}` + `${root_domain}`
- Secrets Manager secret: `ct-gitops/dev/demo-app`
- GitHub Actions OIDC IAM role: `ct-gitops-dev-gha-ecr-push`

## Step 1 — Create terraform.tfvars
```bash
cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev
cp terraform.tfvars.example terraform.tfvars

Edit terraform.tfvars (minimum):

project_slug = "ct-gitops"

environment = "dev"

aws_region = "us-east-2"

root_domain = "p1.<your-domain>" # recommended delegated sub-zone

Optional but recommended:

cluster_endpoint_public_access_cidrs = ["<YOUR_PUBLIC_IP>/32"] (safer than 0.0.0.0/0)

Step 2 — Plan + apply
terraform init
terraform plan
terraform apply
Step 3 — Configure kubectl
aws eks update-kubeconfig --region us-east-2 --name ct-gitops-dev
kubectl get nodes -o wide
kubectl -n kube-system get pods
Step 4 — Capture important outputs
terraform output

Look for:

cluster_name

cluster_oidc_issuer_url

demo_app_repository_url

hosted_zone_id

name_servers

acm_certificate_arn

gha_ecr_push_role_arn

demo_app_secret_arn

✅ Done when:

EKS is ACTIVE; nodes Ready

Terraform outputs show zone NS values, cert ARN, OIDC role ARN


---