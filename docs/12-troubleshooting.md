# 12 — Troubleshooting (battle-tested)

This section is based on real issues encountered in the reference build.

---

## A) Terraform / stack ordering

### A1) `data.aws_eks_cluster` NotFound / Helm provider reads cluster too early
**Symptom**
- `terraform plan/apply` fails in an add-ons stack
- error looks like “cluster not found” or Kubernetes provider can’t connect

**Cause**
- Add-ons stack tries to read the cluster endpoint/CA/auth before the cluster exists.

**Fix (recommended pattern)**
- Split Terraform into:
  - `envs/dev` (infra: VPC + EKS + IRSA outputs)
  - `envs/dev-addons` (Helm/Kubernetes providers + releases)
- Use `terraform_remote_state` in add-ons and add a guard/precondition.

---

## B) Argo CD / Kustomize issues

### B1) Argo app shows Sync = Unknown / repo render error
**Symptom**
- Argo `demo-app` shows `Unknown` or can’t sync
- GitHub Actions `kustomize build` fails too

**Cause**
- `kustomize build` fails (often patch target missing).

**Fix**
- Run locally:
  ```bash
  kustomize build apps/demo/overlays/dev

If you use a strategic merge patch, ensure the base contains the target resource.
Example: overlay patches ConfigMap/demo-placeholder → base must define that ConfigMap.

B2) MalformedYAMLError / indentation breaks manifests

Symptom

Argo events show YAML parse errors:
mapping values are not allowed...

Fix

Run:

kustomize build clusters/ct-gitops-dev

Fix indentation. Re-run build until it succeeds.

B3) Duplicate YAML keys (AppProject whitelist keys)

Symptom

CI fails with:
mapping key "clusterResourceWhitelist" already defined

Cause

YAML has duplicate keys (not allowed).

Fix

Merge duplicates into one list block. Keep each key only once.

B4) Argo stuck retrying an old bad revision

Symptom

You fixed the repo, but Argo keeps retrying the older revision.

Fix

Clear the running operation and force refresh:

In UI: terminate operation, then refresh hard and sync.

Or patch application annotations to refresh.

C) ExternalDNS issues
C1) Helm chart version not found

Symptom

Helm error: chart version not found.

Fix

List versions:

helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo bitnami/external-dns --versions | head

Pin an available version.

C2) ImagePullBackOff (image tag not found)

Symptom

ExternalDNS pod in ImagePullBackOff

image tag doesn’t exist in registry

Fix

Pin a valid image tag

Prefer the official ExternalDNS chart/image for long-term cleanliness

If using Bitnami and tag is missing, switch to a known-good image source.

C3) CrashLoop with unknown long flag '--0'

Symptom

logs show invalid args like --0

Cause

Chart rendered extraArgs incorrectly (list format bug).

Fix

Remove extraArgs list and use chart-native values:

domainFilters, zoneIdFilters, etc.

D) DNS delegation (sub-zone)
D1) SERVFAIL for p1.<domain> or demo.p1.<domain>

Symptom

nslookup returns SERVFAIL

hostname won’t resolve

Cause (common)

Parent zone NS delegation record was entered incorrectly (e.g., pasted as one string).

Fix

In the parent hosted zone:

Create NS record p1

Add the 4 Route 53 name servers as four separate values

Verify:

nslookup -type=NS p1.<domain> 1.1.1.1
E) ACM / HTTPS
E1) HTTPS listener exists but cert not attached / wrong region

Symptom

443 listener missing cert or HTTPS fails

Cause

ACM cert must be in the same region as the ALB (here: us-east-2)

Fix

Request/validate the cert in us-east-2

Update Ingress annotations with correct cert ARN.

F) ESO (External Secrets Operator)
F1) API version mismatch (v1beta1 vs v1)

Symptom

Argo sync fails:
could not find version "v1beta1"... Version "v1" is installed

Fix

Update manifests to:

apiVersion: external-secrets.io/v1
for:

ClusterSecretStore

ExternalSecret

F2) Secret not created in namespace

Symptom

ExternalSecret exists but target Secret missing

Fix

Check ESO controller logs:

kubectl -n external-secrets logs deploy/external-secrets --tail=100

Ensure IRSA policy allows GetSecretValue on the exact secret ARN.

G) CI/CD (GitHub Actions OIDC + PR)
G1) OIDC provider already exists (Terraform 409 conflict)

Symptom

Terraform errors creating OIDC provider:
EntityAlreadyExists / 409

Fix

Import existing provider into state (recommended),
then manage it in Terraform.

G2) AWS creds not loading in workflow

Symptom

configure-aws-credentials step fails

role-to-assume missing

Fix

Add Repo Variable:

AWS_ROLE_ARN = Terraform output role ARN

Ensure workflow uses ${{ vars.AWS_ROLE_ARN }}

G3) GitOps PR step fails with auth prompt

Symptom

could not read Username...

Cause

Token stored in repo variables (not secrets), or not provided to action.

Fix

Store token as repo Secret:

GITOPS_REPO_TOKEN

Use it in PR step.

G4) Regex replacement errors (invalid group reference)

Symptom

workflow step fails: invalid group reference

Fix

Use safe backrefs in replacements:

\g<1> instead of \1 when replacement may interpret \10.

H) Endpoint deprecation warning

Symptom

Warning about Endpoints being replaced by EndpointSlice

Fix

Use EndpointSlice:

kubectl -n demo get endpointslices -l kubernetes.io/service-name=demo-app -o wide

---