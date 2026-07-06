# Conventions

Single-operator repo; these conventions exist so future-me doesn't have to
re-derive them.

## Fork model (`laird-forks` org)

- **Detached private forks**: GitHub can't make a private fork of a public
  repo, so `tools/fork-upstream` (in ansible-homelab) creates a fresh
  private repo with upstream history pushed in and an `upstream` remote.
- **Branches**: `main` tracks upstream verbatim — never commit to it.
  Patches live on `local`, rebased onto upstream tags.
- **Tags**: `vX.Y.Z-local.N` — upstream version + localization revision.
  This repo pins those tags; a fork update is an explicit ref bump here.
- **Naming**: upstream name verbatim (`laird-forks/immich`, not
  `immich-fork`).
- **Visibility**: private by default. Public is an opt-in flip via
  `tools/publish-fork`, which runs the sanitization ceremony (gitleaks,
  fingerprint grep, human log review) first.
- Every fork carries a `LOCAL-CHANGES.md` on `local` listing each patch
  and whether it's upstreamable.

## This repo

- One PR (or commit) per workload change, with an observation window
  before the next.
- `projects/<x>/UPSTREAM.md` records the upstream source, tracking method
  (fork tag / remote base / vendored), and current pinned ref.
- Secrets are SOPS-encrypted with the age key in `.sops.yaml` — files
  named `*.sops.yaml`. Nothing else may contain credentials.
- Renovate watches this repo and `laird-forks/*` for upstream bumps
  (`.github/renovate.json`).
