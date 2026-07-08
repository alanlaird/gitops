# o11y — upstream tracking

App-of-apps observability bundle, ported from `ansible-homelab:terraform/talos4/o11y/`
(the live talos4 install) on 2026-07-08. Shape 3 platform bundle.

All components are upstream Helm charts pinned by `targetRevision` in
`base/applications/*.yaml` — remote-base tracking, no forks yet.

| Component | Chart | Pinned |
|-----------|-------|--------|
| kube-prometheus-stack | prometheus-community | 72.3.0 |
| loki-stack | grafana | 2.10.3 |
| tempo | grafana | 1.24.4 |
| phlare | grafana | 0.5.4 (renamed upstream to pyroscope — migrate when touched) |
| victoria-metrics-operator | victoriametrics | 0.59.0 |
| prometheus-adapter | prometheus-community | 5.3.0 |
| metallb (ingress component) | metallb | 0.15.3 |
| cert-manager (ingress component) | jetstack | v1.17.2 |
| ingress-nginx (ingress component) | kubernetes | 4.12.1 |

Layout:
- `base/` — cluster-neutral core telemetry (no LB IPs, no ingress hosts, no
  CP endpoint IPs). Every overlay must patch: promtail cluster label,
  kube-prometheus CP endpoints, storage/retention.
- `components/ingress/` — kustomize Component adding MetalLB + cert-manager +
  ingress-nginx + LB Services. Only overlays on clusters with a MetalLB pool
  include it.
- `overlays/<cluster>/` — per-cluster values patches and glue overrides.
