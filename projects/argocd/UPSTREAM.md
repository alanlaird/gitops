# argocd — upstream tracking

Self-managing ArgoCD: each cluster's Argo is bootstrapped once by hand
(`terraform/<cluster>/k8s/argocd/deploy` in ansible-homelab, which also
seeds the sops-age + repo-credential Secrets from the ansible vault), then
adopts itself via the `argocd` Application in `clusters/<cluster>/apps.yaml`.
Upgrades from then on are a `targetRevision` bump here.

- Chart: `argo-cd` from https://argoproj.github.io/argo-helm (remote base, no fork)
- Values: `values-<cluster>.yaml`, referenced via multi-source `$values`
- The bootstrap Secrets are deliberately NOT in git — they carry the age key
  and PAT, and exist before Argo does.
