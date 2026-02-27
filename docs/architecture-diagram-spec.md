# Architecture diagram spec (Eraser.io)

You’re using **Eraser**. Eraser supports both:
- Mermaid diagrams
- Eraser “diagram-as-code” DSL

This guide includes both.

## Files
- Mermaid: `docs/architecture-mermaid.md`
- Eraser DSL: `docs/ct-gitops-v2-architecture.eraserdiagram`

---

## One-page diagram rules (for a clean portfolio visual)

### Canvas
- 16:9, landscape
- 3 vertical lanes:
  1) GitHub / CI
  2) AWS services
  3) EKS cluster

### Keep it simple
- 10–12 nodes max
- 8–12 arrows max
- clear labels (no tiny text)

### Must-show concepts (what recruiters care about)
- **OIDC** (no AWS keys)
- **PR gate** for deploy (GitOps)
- **Argo CD** as reconciler (self-heal + prune)
- **ALB Ingress** + **ExternalDNS** + **ACM**
- **Secrets Manager + ESO** (no secrets in Git)

---

## Eraser diagram-as-code tips

How to use:
1) In Eraser → Insert → Diagram-as-code → Architecture
2) Paste the contents of `ct-gitops-v2-architecture.eraserdiagram`
3) Export PNG for LinkedIn + SVG for website