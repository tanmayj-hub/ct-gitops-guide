```mermaid
flowchart LR
  %% -----------------------------
  %% GitHub (source of truth)
  %% -----------------------------
  subgraph GH["GitHub"]
    InfraRepo["gitops-eks-infra\nTerraform (envs/dev + envs/dev-addons)"]
    GitOpsRepo["gitops-eks-apps\nKustomize + Argo Applications"]
    AppRepo["ct-gitops-demo-app\nApp source + CI workflow"]
    GHA["GitHub Actions\nBuild → Push → PR"]
  end

  %% -----------------------------
  %% AWS services
  %% -----------------------------
  subgraph AWS["AWS (us-east-2)"]
    IAM["IAM OIDC Role\n(ct-gitops-dev-gha-ecr-push)"]
    ECR["ECR\nct-gitops/demo-app"]
    R53["Route 53\np1.<domain> zone"]
    ACM["ACM\nWildcard cert (*.p1.<domain>)"]
    SM["Secrets Manager\nct-gitops/dev/demo-app"]
    ALB["ALB\n(created from Ingress)"]
  end

  %% -----------------------------
  %% EKS runtime
  %% -----------------------------
  subgraph EKS["EKS: ct-gitops-dev"]
    Argo["Argo CD\n(app-of-apps)"]
    LBC["AWS Load Balancer Controller"]
    ExtDNS["ExternalDNS"]
    ESO["External Secrets Operator"]
    Demo["demo-app\nDeployment + Service + Ingress"]
  end

  Users["Users\nBrowser"]

  %% Infra provisioning + add-ons install
  InfraRepo -->|"terraform apply\n(VPC/EKS/IRSA/ECR/Route53/ACM/Secrets/OIDC)"| AWS
  InfraRepo -->|"terraform apply\n(Helm add-ons + root Argo App)"| EKS

  %% CI/CD to GitOps
  AppRepo -->|"push to main"| GHA
  GHA -->|"OIDC AssumeRole\n(no AWS keys)"| IAM
  GHA -->|"docker push\n0.1.X / sha-*"| ECR
  GHA -->|"PR: bump images[].newTag"| GitOpsRepo

  %% GitOps reconcile
  GitOpsRepo -->|"desired state"| Argo
  Argo -->|"apply + self-heal + prune"| Demo

  %% Secrets flow
  SM -->|"GetSecretValue (IRSA)"| ESO
  ESO -->|"creates K8s Secret"| Demo

  %% Ingress / DNS / TLS flow
  Demo -->|"Ingress"| LBC
  LBC -->|"provisions"| ALB
  ExtDNS -->|"creates A/AAAA + TXT"| R53
  R53 -->|"DNS resolves hostname"| Users
  ACM -->|"TLS cert attached"| ALB
  ALB -->|"HTTPS"| Users

---