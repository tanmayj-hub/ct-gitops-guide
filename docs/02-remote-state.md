# 02 — Remote Terraform state (S3 + DynamoDB)

The infra repo includes a bootstrap stack:
- `gitops-eks-infra/bootstrap/tfstate`

It creates:
- S3 bucket (Terraform state)
- DynamoDB table (state locking)

## Step 1 — Verify AWS auth
```bash
aws sts get-caller-identity
Step 2 — Bootstrap apply (local state)
cd ~/projects/ct-gitops/gitops-eks-infra/bootstrap/tfstate

terraform init
terraform plan
terraform apply
Step 3 — Initialize infra with remote backend
cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev
terraform init -reconfigure
terraform plan

✅ Done when:

terraform init confirms the S3 backend

the state object exists in S3

the lock table exists in DynamoDB


---