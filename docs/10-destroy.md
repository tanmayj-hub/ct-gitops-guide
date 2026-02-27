# 10 — Destroy safely (stop costs)

Destroy order matters (ALB + NAT + EKS can keep charging).

## Before destroy: capture proof
Use the screenshot checklist in `assets/screenshots/README.md`.

## Step 1 — Remove Ingress (GitOps) to delete ALB
ExternalDNS is configured with `upsert-only`, so records might remain; ALB must be removed first.

In `gitops-eks-apps`:
- remove `apps/demo/overlays/dev/ingress.yaml` (or remove it from kustomization)

Commit/push:
```bash
cd ~/projects/ct-gitops/gitops-eks-apps
git add -A
git commit -m "chore: remove demo ingress before teardown"
git push

Wait for Argo to sync and ALB to delete:

aws elbv2 describe-load-balancers --output table
Step 2 — Destroy add-ons stack
cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev-addons
terraform destroy
Step 3 — Destroy infra stack
cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev
terraform destroy
Step 4 — Route 53 cleanup

If you used a delegated zone p1.<domain>:

Delete leftover demo.p1... records in the p1 zone (A/AAAA/TXT).

Delete the hosted zone p1.<domain>.

Remove the NS delegation record p1 from the parent zone.

Step 5 — Optional: delete remote state resources

Only do this if you want a completely clean account:

empty the S3 state bucket:

aws s3 rm s3://<STATE_BUCKET> --recursive

delete bucket and lock table:

aws s3 rb s3://<STATE_BUCKET>
aws dynamodb delete-table --table-name <LOCK_TABLE>

---
