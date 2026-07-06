# gitops

GitOps monorepo for the homelab Talos fleet. Each cluster runs its own
ArgoCD, scoped to `clusters/<self>/apps.yaml`; workloads are defined once
under `projects/` with per-cluster overlays.

Authoritative design and migration plan: `readme/gitops-migration.md` in
ansible-homelab. Summary below.

## Fleet

| Cluster        | Role                  | Notes |
|----------------|-----------------------|-------|
| **taloshw**    | prod pair — hardware  | Bare-metal OptiPlex; NFS to agate; MetalLB on LAN |
| **talos4**     | prod pair — VM        | The expansion lever — grow workers as capacity demands |
| **talos5154**  | ephemeral test        | Normally powered off; stood up to test big changes |

Cluster names are node/address blocks (talos4 = talos41–44 at
172.16.16.4x), not roles. Roles float.

## Prod-pair model

taloshw and talos4 are production peers running the identical pattern.

- Moving a workload = flipping which cluster's `apps.yaml` claims it.
- Cluster upgrades: drain workloads to the peer, upgrade, move back.
- Pair-mobile workloads use NFS-backed PVs (agate) reachable from both
  clusters; node-local storage anchors a workload and needs justification.
- Internet-facing apps are a **namespace** boundary, not a cluster
  boundary: dedicated namespaces, default-deny NetworkPolicies,
  per-cluster cloudflared so tunnels follow workloads.

## Layout

```
clusters/<cluster>/apps.yaml   # what this cluster runs (Argo Applications)
projects/<name>/
  UPSTREAM.md                  # where it comes from, how it's tracked
  base/                        # shared manifests / chart wiring
  overlays/<cluster>/          # env specifics: storage class, IPs, hostnames
```

## Three-layer ownership

| Layer           | Lives at                                  | Holds |
|-----------------|-------------------------------------------|-------|
| Upstream        | original repo                             | What we track |
| Fork            | `laird-forks/<name>`                      | Generic patches — PR-able back |
| Cluster overlay | `projects/<x>/overlays/<cluster>/`        | Env specifics |

**Fork vs overlay test:** *would this patch make sense in someone else's
homelab?* Yes → fork. No → overlay. Fork eagerly — any patch beyond an
overlay gets a fork, even one commit deep. See CONTRIBUTING.md for
conventions.

## Tiers for new projects

1. **Throwaway** — manual `kubectl apply` on the test cluster; no repo entry.
2. **Vendored experiment** — upstream copied into `projects/<x>/base/`,
   edited freely; claimed only by the test cluster.
3. **Formalized** — upstream-tracking via fork or remote base; overlays
   per cluster; Renovate watching.

## Secrets

SOPS + age. Public key lives in `.sops.yaml`; the private key is held in
ansible-homelab's vault (`gitops_age_key`) and applied as a Secret in each
cluster's `argocd` namespace at bootstrap. No plaintext secrets in this
repo, ever.
