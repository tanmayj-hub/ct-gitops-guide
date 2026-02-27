# 07 — Public Ingress + DNS + HTTPS (ALB + ExternalDNS + Route 53 + ACM)

This stage exposes the demo app publicly with a real hostname and TLS:

- Ingress → ALB (AWS Load Balancer Controller)
- ExternalDNS → Route 53 records (A/AAAA + TXT)
- ACM wildcard cert attached to ALB → HTTPS + redirect

## Step 1 — Route 53 hosted zone (Terraform infra stack)
The infra stack manages a hosted zone defined by:
- `root_domain` (default: `p1.cloudwithtanmay.com`)

Apply:
```bash
cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev
terraform apply
terraform output name_servers
terraform output hosted_zone_id
Step 2 — Delegate the sub-zone from your parent domain

If your parent domain is already in Route 53:

go to cloudwithtanmay.com hosted zone

create an NS record:

Name: p1

Values: the 4 name servers from output (each as a separate value)

Verify:

nslookup -type=NS p1.<your-domain> 1.1.1.1
Step 3 — ExternalDNS (Terraform add-ons stack)

ExternalDNS is installed in kube-system and configured with:

domainFilters: [p1.<your-domain>]

zoneIdFilters: [<your zone id>]

policy: upsert-only (safe for demos)

Verify logs:

kubectl -n kube-system logs deploy/external-dns --tail=120
Step 4 — Ingress (GitOps)

Ingress file:

gitops-eks-apps/apps/demo/overlays/dev/ingress.yaml

You MUST edit:

external-dns.alpha.kubernetes.io/hostname

spec.rules.host
to match your domain.

Commit/push and wait for Argo sync:

cd ~/projects/ct-gitops/gitops-eks-apps
git add -A
git commit -m "feat(demo): add ALB ingress + external-dns hostname"
git push

Verify ALB address:

kubectl -n demo get ingress demo-app -o wide
Step 5 — HTTPS (ACM wildcard cert + redirect)

Infra stack creates ACM wildcard:

*.${root_domain} plus SAN ${root_domain}

Get cert ARN:

cd ~/projects/ct-gitops/gitops-eks-infra/envs/dev
terraform output acm_certificate_arn

Update Ingress annotations in ingress.yaml:

alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'

alb.ingress.kubernetes.io/certificate-arn: "<YOUR_ACM_ARN>"

alb.ingress.kubernetes.io/ssl-redirect: "443"

Commit/push and verify:

curl -I http://demo.<your root_domain>
curl -I https://demo.<your root_domain>

✅ Done when:

hostname resolves to the ALB

HTTP redirects to HTTPS

browser shows valid TLS


---