```mermaid
flowchart LR
  Dev[Developer] -->|push to main| AppRepo[ct-gitops-demo-app]
  AppRepo -->|GitHub Actions| GHA[CI: Build + Push + PR]
  GHA -->|OIDC AssumeRole| IAMRole[IAM Role: gha-ecr-push]
  IAMRole --> AWS[(AWS)]

  GHA -->|docker push| ECR[ECR: ct-gitops/demo-app]
  GHA -->|PR bump image tag| GitOpsRepo[gitops-eks-apps]

  GitOpsRepo -->|desired state| Argo[Argo CD (app-of-apps)]
  Argo -->|apply| EKS[EKS Cluster]

  SecretsMgr[Secrets Manager] --> ESO[External Secrets Operator]
  ESO -->|creates K8s Secret| EKS

  EKS -->|Ingress| LBC[AWS Load Balancer Controller]
  LBC --> ALB[ALB (internet-facing)]
  ALB -->|HTTPS (ACM)| Users[Users]

  ExternalDNS --> Route53[Route 53 Hosted Zone]
  Route53 --> Users

---