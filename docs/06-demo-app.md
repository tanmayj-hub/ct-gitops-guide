# 06 — Demo app deploy (ECR + GitOps)

The demo app:
- Container image stored in ECR (`ct-gitops/demo-app`)
- Deployed by Argo CD from `gitops-eks-apps`
- Exposed publicly later via Ingress/ALB

## Step 1 — Confirm ECR repo exists
```bash
aws ecr describe-repositories --region us-east-2 --repository-names ct-gitops/demo-app
Step 2 — Build + push image manually (first time)

This is a one-time “manual baseline”. CI will automate later.

cd ~/projects/ct-gitops/ct-gitops-demo-app
docker build -t ct-gitops-demo-app:local .

AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.us-east-2.amazonaws.com"
ECR_REPO="${ECR_REGISTRY}/ct-gitops/demo-app"

aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin "${ECR_REGISTRY}"

docker tag ct-gitops-demo-app:local "${ECR_REPO}:0.1.0"
docker push "${ECR_REPO}:0.1.0"
Step 3 — Ensure GitOps overlay references your image

File:

gitops-eks-apps/apps/demo/overlays/dev/kustomization.yaml

It should set images[].newName to your ECR repo and newTag to 0.1.0.

Commit/push:

cd ~/projects/ct-gitops/gitops-eks-apps
git add -A
git commit -m "feat(demo): deploy demo app"
git push
Step 4 — Verify
kubectl -n demo get deploy,svc,pods -o wide
kubectl -n demo rollout status deploy/demo-app
kubectl -n demo get pod -o jsonpath='{.items[0].spec.containers[0].image}{"\n"}'

Local test:

kubectl -n demo port-forward svc/demo-app 8081:80
# open http://localhost:8081

✅ Done when:

demo-app Application is Synced/Healthy

deployment is Available and using ECR image tag


---