# 04 — Add-ons stack (LBC + Argo CD + ExternalDNS + ESO) via Terraform + Helm

This stage applies:
- `gitops-eks-infra/envs/dev-addons`

Why a separate stack?
- Kubernetes/Helm providers need the cluster to exist.
- Separating avoids “cluster not found yet” timing issues.

## What this stack installs (reference versions)
- AWS Load Balancer Controller: chart `aws-load-balancer-controller` **3.0.0** (IRSA)
- Argo CD: chart `argo-cd` **9.4.3** (app v3.3.1), private (ClusterIP)
- ExternalDNS: chart `external-dns` **9.0.3** (IRSA, scoped filters)
- External Secrets Operator: chart `external-secrets` **2.0.1** (IRSA)

## Step 1 — Configure terraform.tfvars
```bash
cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev-addons
cp terraform.tfvars.example terraform.tfvars

Defaults in this stack assume infra state:

bucket: ct-gitops-tfstate-us-east-2

key: ct-gitops/dev/terraform.tfstate

Step 2 — Plan + apply
terraform init -reconfigure
terraform plan
terraform apply
Step 3 — Verify installs
Load Balancer Controller
kubectl -n kube-system get deployment aws-load-balancer-controller
kubectl -n kube-system get pods | grep aws-load-balancer-controller || true
Argo CD (private)
kubectl -n argocd get pods
kubectl -n argocd get svc argocd-server -o wide

Port-forward:

kubectl -n argocd port-forward svc/argocd-server 8080:443
# open https://localhost:8080
ExternalDNS
kubectl -n kube-system get deploy external-dns
kubectl -n kube-system logs deploy/external-dns --tail=80
External Secrets Operator
kubectl -n external-secrets get pods
Common issues (from the reference build)

ExternalDNS chart version not found → use a version that exists in repo (reference used 9.0.3)

ExternalDNS image tag not found → the reference build used bitnamilegacy/external-dns as a workaround

ExternalDNS extraArgs rendering weird flags (--0) → prefer chart-native zoneIdFilters/domainFilters

✅ Done when:

all controller pods are Running/Ready

Argo UI is accessible via port-forward


---