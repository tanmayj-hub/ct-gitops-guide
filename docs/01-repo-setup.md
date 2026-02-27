# 01 — Repo setup (fork + clone + required edits)

You will fork 3 repos:

1) Terraform infra: `gitops-eks-infra`
2) GitOps manifests: `gitops-eks-apps`
3) App source + CI: `ct-gitops-demo-app`

## Step 1 — Fork
Fork each repo from Tanmay’s GitHub into your account.

## Step 2 — Clone (Git Bash)
```bash
mkdir -p ~/projects/ct-gitops && cd ~/projects/ct-gitops

git clone https://github.com/<YOUR_GITHUB>/gitops-eks-infra.git
git clone https://github.com/<YOUR_GITHUB>/gitops-eks-apps.git
git clone https://github.com/<YOUR_GITHUB>/ct-gitops-demo-app.git
Step 3 — REQUIRED: update hardcoded repo URLs to your fork

Argo CD Applications point to the GitOps repo via repoURL.
If you don’t change these, Argo will try to sync Tanmay’s repos.

A) Infra repo — root Argo Application

Edit:

gitops-eks-infra/envs/dev-addons/10-gitops-bootstrap.tf

Change:

repoURL: https://github.com/tanmayj-hub/gitops-eks-apps.git
→ to your fork:

repoURL: https://github.com/<YOUR_GITHUB>/gitops-eks-apps.git

B) Apps repo — child Applications

Edit:

gitops-eks-apps/clusters/ct-gitops-dev/argocd/applications/*.yaml

Change each repoURL from tanmayj-hub/... to <YOUR_GITHUB>/....

Commit and push both repos:

cd ~/projects/ct-gitops/gitops-eks-infra
git add -A
git commit -m "chore: update gitops repoURL to my fork"
git push

cd ~/projects/ct-gitops/gitops-eks-apps
git add -A
git commit -m "chore: update repo URLs for my fork"
git push
Step 4 — Domain plan (recommended)

The reference project uses a delegated sub-zone:

p1.<your-domain>

Example:

parent: cloudwithtanmay.com

delegated zone: p1.cloudwithtanmay.com

host: demo.p1.cloudwithtanmay.com

This keeps cleanup safe and allows more sub-projects later.

If you don’t want DNS:

you can skip docs/07-ingress-dns-tls.md and access via ALB DNS name.

Step 5 — Prepare GitHub token for cross-repo PR (Stage 9)

In Stage 9, the app repo’s workflow opens a PR to the GitOps repo.
You’ll create a fine-grained PAT with access ONLY to gitops-eks-apps:

Contents: Read/Write

Pull requests: Read/Write

Store it as a repo secret in ct-gitops-demo-app:

GITOPS_REPO_TOKEN


---