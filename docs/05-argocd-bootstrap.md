# 05 — GitOps bootstrap (App-of-Apps)

Goal:
- Terraform creates **ONE root Argo CD Application**
- Root app syncs the cluster folder from `gitops-eks-apps`
- Child apps (defined in Git) then deploy:
  - namespaces
  - AppProject
  - demo app resources (and later more apps)

## What “app-of-apps” means (simple)
- Root Application points to `clusters/ct-gitops-dev`
- That folder contains other Applications
- Argo applies those Applications, and they apply everything else

## Step 1 — Apply root app (Terraform add-ons stack)
The root Application is created in:
- `gitops-eks-infra/envs/dev-addons/10-gitops-bootstrap.tf`

Apply:
```bash
cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev-addons
terraform apply
Step 2 — Verify Argo sees apps
kubectl -n argocd get applications

Expected:

ct-gitops-dev-root

cluster-bootstrap

demo-app

Step 3 — Common pitfall: Kustomize build failures

If kustomize build fails, Argo cannot render manifests and may show Sync = Unknown.

Validate locally:

cd ~/projects/ct-gitops/gitops-eks-apps
kustomize build clusters/ct-gitops-dev >/dev/null
kustomize build apps/demo/overlays/dev >/dev/null
echo "kustomize builds OK"

✅ Done when:

All Argo Applications are Synced/Healthy

Namespace demo exists (created by cluster-bootstrap)


---