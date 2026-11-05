# project-atmos-onprem-gitops

Desired state of the on-premise Kubernetes cluster (k3s) of Project Atmos, delivered by Flux.

## Purpose

Flux watches this repository and applies it to the cluster. Flux image automation polls the Nexus registry and commits new image tags to `main`; Flux then upgrades the Helm release.

## Planned content

- Flux configuration (sources, kustomizations, image automation)
- Helm chart of the application (frontend, backend, MongoDB, Redis)
- Ingress (`ingressClassName: traefik`) and the Traefik `HelmChartConfig`
- NetworkPolicy
- Monitoring: Prometheus, Grafana, kube-state-metrics (namespace `monitoring`)
- Values

## Secrets

Application secrets (OpenWeather key, MongoDB password) are Kubernetes Secrets created by hand and never stored in Git (see `.gitignore`). Flux uses a deploy key with write access to this repository only.

## Related repositories

- [project-atmos-dev](https://github.com/Kernel-Club/project-atmos-dev) — application and architecture docs (`docs/on-premise/`)
- [project-atmos-onprem-provision](https://github.com/Kernel-Club/project-atmos-onprem-provision) — Ansible for the VMs and k3s
- [project-atmos-onprem-jenkins](https://github.com/Kernel-Club/project-atmos-onprem-jenkins) — CI pipeline that publishes the images

## Branching

`main` is the state of the cluster. Changes go through `feature/*` or `fix/*` branches and a Pull Request to `main`. Flux commits image tag updates directly to `main`. See `docs/on-premise/branching-model.md` in `project-atmos-dev`.
