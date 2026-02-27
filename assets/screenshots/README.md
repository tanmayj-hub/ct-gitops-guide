# Screenshot checklist (proof set)

Use 8–10 screenshots max. This order tells a clean story.

## A) Build/Deploy story (must-have)
1) **Architecture diagram** (Eraser export PNG)
2) **GitHub Actions run** in `ct-gitops-demo-app` (green): shows Build → Push → PR
3) **PR in `gitops-eks-apps`** that bumps `images[].newTag` (before merge)
4) **Argo CD UI**: `ct-gitops-dev-root`, `cluster-bootstrap`, `demo-app` = Synced/Healthy
5) **ECR repo** showing tags (e.g., `0.1.X`, `sha-...`)
6) **Ingress output**:
   - `kubectl -n demo get ingress demo-app -o wide` (shows ALB DNS)
7) **Route 53 records** for `demo.p1...`:
   - A/AAAA alias
   - TXT ownership records created by ExternalDNS
8) **Browser** on `https://demo.p1...`:
   - lock icon visible
   - page displays Build tag/sha (prove rollout)

## B) Security story (nice-to-have)
9) **ESO proof**:
   - `kubectl get clustersecretstore aws-secretsmanager` (Valid=True)
   - `kubectl -n demo get externalsecret demo-app-secret` (SecretSynced=True)
   - or `kubectl -n demo exec ... env | grep WELCOME_MESSAGE`
10) **OIDC proof**:
   - Terraform output showing role ARN
   - GitHub Action step “Configure AWS credentials (OIDC)”

## Tip
Crop screenshots so sensitive IDs aren’t front-and-center (account ID, cert ARN).