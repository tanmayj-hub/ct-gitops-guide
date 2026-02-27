# 08 — Secrets baseline (Secrets Manager + External Secrets Operator)

Goal:
- secret values live in AWS Secrets Manager
- ESO syncs them to Kubernetes Secret
- App reads env var from that Secret
- no secret values are stored in Git

## Step 1 — Secrets Manager secret (Terraform infra stack)
Infra creates:
- secret name: `ct-gitops/dev/demo-app`
- JSON payload includes `WELCOME_MESSAGE`

Apply/verify outputs:
```bash
cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev
terraform apply
terraform output demo_app_secret_name
terraform output demo_app_secret_arn
Step 2 — ESO install (Terraform add-ons stack)

Apply:

cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev-addons
terraform apply
kubectl -n external-secrets get pods
Step 3 — GitOps manifests

Files in GitOps repo:

ClusterSecretStore (cluster-level): clusters/ct-gitops-dev/...

ExternalSecret (demo namespace): apps/demo/overlays/dev/external-secret.yaml

Deployment consumes secret env var: overlay patch

Important:

Use apiVersion external-secrets.io/v1 (the reference build uses v1)

Verify:

kubectl get clustersecretstore aws-secretsmanager
kubectl -n demo get externalsecret demo-app-secret
kubectl -n demo get secret demo-app-secrets
kubectl -n demo exec deploy/demo-app -- env | grep WELCOME_MESSAGE
Common issues (from the reference build)

apiVersion mismatch (v1beta1 vs v1) → update manifests to external-secrets.io/v1

Argo stuck on old revision → clear running operation and refresh/sync

✅ Done when:

ClusterSecretStore Valid=True

ExternalSecret SecretSynced=True

app prints WELCOME_MESSAGE


---