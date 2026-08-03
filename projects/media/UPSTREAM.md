# media — upstream tracking

Hand-rolled manifests, no upstream to track — fork model N/A. Ported from
the fir podman pet (`ansible-homelab:roles/media/`) starting 2026-08. Shape 1
project (per `ansible-homelab:readme/gitops-migration.md`'s "single
chart/manifest set" tier).

Apps: Bazarr, Prowlarr, Radarr, Sonarr, SABnzbd, Tubesync. Each runs the
`lscr.io/linuxserver/<app>:latest` (or `ghcr.io/meeb/tubesync:latest`)
upstream image directly — no chart, no fork, just a Deployment/Service/PVC
set per app.

Config for Sonarr/Radarr/Prowlarr is reconciled declaratively by a
`buildarr` Deployment (see `base/buildarr/`) rather than hand-edited —
`callum027/buildarr`, an individual-maintainer image, not org-backed.
Bazarr and SABnzbd have their own small reconciliation CronJobs using their
native settings APIs (no equivalent third-party tool exists for either).

Layout:
- `base/` — namespace, KSOPS secrets, the 7 static NFS-CSI PVs unioning
  ophir/bullards (`pv-library.yaml`, cluster-agnostic since both NAS units
  are LAN-wide reachable), and one directory per app/reconciler.
- `overlays/taloshw/` — MetalLB Service IP patches. `overlays/talos4/` not
  scaffolded yet — add only when Phase 5.6's mobility drill needs it.
