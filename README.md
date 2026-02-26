# ct-gitops v2 — GitOps on EKS with Terraform + Argo CD + Kustomize + CI/CD

This repo is a **beginner-friendly, step-by-step guide** to replicate my GitOps v2 project:

- **Terraform** provisions AWS infra (VPC, EKS, IAM/IRSA, ECR, Route53, ACM, Secrets Manager)
- **Argo CD** reconciles desired state from Git (app-of-apps)
- **Kustomize** manages environment overlays (dev)
- **GitHub Actions (OIDC)** builds & pushes images to **ECR**
- Pipeline opens a **PR to GitOps repo** updating the image tag → merge triggers Argo deploy

## Repos you will fork

Fork these repos into your GitHub account:

1) Infra (Terraform): https://github.com/tanmayj-hub/gitops-eks-infra  
2) GitOps (Kustomize + Argo Apps): https://github.com/tanmayj-hub/gitops-eks-apps  
3) App source (CI build): https://github.com/tanmayj-hub/ct-gitops-demo-app  

## Architecture (high level)

```mermaid
flowchart LR
  Dev[Developer] -->|push| GHA[GitHub Actions]
  GHA -->|OIDC AssumeRole| AWS[(AWS)]
  GHA -->|build+push| ECR[ECR: ct-gitops/demo-app]
  GHA -->|PR: bump image tag| GitOpsRepo[gitops-eks-apps]

  GitOpsRepo -->|sync| Argo[Argo CD]
  Argo -->|apply manifests| EKS[EKS Cluster]
  EKS --> ALB[ALB via AWS LBC]
  ALB -->|HTTPS| Users[Users]
  ExternalDNS[ExternalDNS] --> Route53[Route 53]
  Route53 --> Users
  SecretsMgr[Secrets Manager] --> ESO[External Secrets Operator]
  ESO --> EKS
Your values (edit these)

AWS region: us-east-2

Project slug: ct-gitops

Env: dev

Cluster name: ct-gitops-dev

Domain: cloudwithtanmay.com

Delegated zone used here: p1.cloudwithtanmay.com

Demo URL: https://demo.p1.cloudwithtanmay.com

Step-by-step stages
Stage 0 — Repo setup & conventions

Follow: docs/01-repo-setup.md

Stage 1 — Tooling

Follow: docs/00-prereqs.md

Stage 2 — Remote Terraform state (S3 + DynamoDB)

Follow: docs/02-remote-state.md

Stage 3 — VPC + EKS (Terraform)

Follow: docs/03-eks-infra.md

Stage 4 — Add-ons (Terraform dev-addons stack)

Installs:

AWS Load Balancer Controller (IRSA)

Argo CD (Helm)

ExternalDNS (IRSA)

External Secrets Operator (IRSA)

Follow: docs/04-addons.md + docs/05-argocd-bootstrap.md

Stage 5–7 — GitOps bootstrap + demo app

Follow: docs/05-argocd-bootstrap.md + docs/06-demo-app.md

Stage 8–9 — Public ingress + DNS + HTTPS (ACM wildcard)

Follow: docs/07-ingress-dns-tls.md

Stage 10 — Secrets baseline (Secrets Manager + ESO)

Follow: docs/08-secrets-eso.md

Stage 13 — CI/CD (OIDC → ECR → GitOps PR → Argo deploy)

Follow: docs/09-cicd-oidc-gitops-pr.md

Destroy (stop costs)

Follow: docs/10-destroy.md

Screenshots checklist (for proof)

See assets/screenshots/README.md (or the LinkedIn section in this chat).