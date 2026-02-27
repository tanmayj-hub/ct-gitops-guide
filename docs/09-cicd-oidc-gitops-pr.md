# 09 — CI/CD (OIDC → ECR → GitOps PR → Argo deploy)

Pipeline design:
- Push to `ct-gitops-demo-app/main`
- GitHub Actions assumes an AWS role using **OIDC** (no AWS keys)
- Build + push image to ECR:
  - `0.1.<run_number>`
  - `sha-<commit sha>`
- Workflow opens a PR to `gitops-eks-apps` bumping `images[].newTag`
- Merge PR → Argo deploys

## Step 1 — Terraform creates OIDC role (infra stack)
Apply:
```bash
cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev
terraform apply
terraform output gha_ecr_push_role_arn

If the GitHub OIDC provider already exists in your AWS account, Terraform may fail with a 409.
Fix pattern:

import the existing provider into state (see infra runbook notes / commit history)

Step 2 — Configure ct-gitops-demo-app repo variables/secrets

In GitHub repo settings (ct-gitops-demo-app):

Repository Variable

AWS_ROLE_ARN = terraform output gha_ecr_push_role_arn

Repository Secret

GITOPS_REPO_TOKEN = fine-grained PAT for gitops-eks-apps

Contents: Read/Write

Pull requests: Read/Write

Step 3 — Confirm workflow file

Workflow path:

.github/workflows/release.yml

It uses:

aws-actions/configure-aws-credentials@v4 with role-to-assume: ${{ vars.AWS_ROLE_ARN }}

peter-evans/create-pull-request@v6 with token: ${{ secrets.GITOPS_REPO_TOKEN }}

Step 4 — Trigger a release (push commit)

Edit index.html to change visible text, then:

cd ~/projects/ct-gitops/ct-gitops-demo-app
git add -A
git commit -m "feat: trigger release"
git push
Step 5 — Verify

GitHub Actions run is green

ECR shows new tags

PR exists in gitops-eks-apps bumping newTag

Merge PR → Argo deploys

Cluster verify:

kubectl -n demo rollout status deploy/demo-app
kubectl -n demo get pod -o jsonpath='{.items[0].spec.containers[0].image}{"\n"}'

Browser verify:

refresh https://demo.<your root_domain> and confirm build version changed

Common issues (from the reference build)

AWS creds step fails → AWS_ROLE_ARN variable missing

GitOps checkout auth fails → PAT stored as VAR not SECRET (must be secret)

Regex replacement errors in workflow → use safe backrefs (\g<1>)

✅ Done when:

commit → image pushed → PR → merge → Argo deploy


---