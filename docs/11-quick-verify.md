# 11 — Quick verify (10 commands)

Run these after you finish replication (or before you take screenshots).

> Assumes:
> - kubeconfig points to `ct-gitops-dev`
> - your demo hostname is `demo.p1.<domain>` (adjust if different)

## 1) Cluster access
```bash
kubectl config current-context
kubectl get nodes -o wide
2) Argo CD apps (GitOps health)
kubectl -n argocd get applications

Expected: ct-gitops-dev-root, cluster-bootstrap, demo-app are Synced/Healthy.

3) Demo app resources
kubectl -n demo get deploy,svc,pods -o wide
kubectl -n demo rollout status deploy/demo-app
4) Confirm running image tag
kubectl -n demo get pod -l app.kubernetes.io/name=demo-app \
  -o jsonpath='{.items[0].spec.containers[0].image}{"\n"}'
5) Ingress + ALB
kubectl -n demo get ingress demo-app -o wide

Expected: ADDRESS populated with an ALB DNS name.

6) DNS resolution
nslookup demo.p1.<domain> 1.1.1.1
7) HTTPS redirect + success
curl -I http://demo.p1.<domain>
curl -I https://demo.p1.<domain>

Expected: HTTP redirects (301/302) to HTTPS; HTTPS returns 200.

8) ExternalDNS health
kubectl -n kube-system get deploy external-dns
kubectl -n kube-system logs deploy/external-dns --tail=30
9) ESO / Secrets sync
kubectl get clustersecretstore aws-secretsmanager
kubectl -n demo get externalsecret demo-app-secret
10) Secret value visible inside pod
kubectl -n demo exec deploy/demo-app -- env | grep WELCOME_MESSAGE

Expected: prints the value stored in AWS Secrets Manager.

✅ If all commands pass, the system is working end-to-end.

---