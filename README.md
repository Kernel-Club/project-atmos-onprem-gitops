# project-atmos-onprem-k3s

Everything that runs inside the on-premise Kubernetes cluster of **Project Atmos**, delivered by Flux (GitOps).

```text
Nexus ─► Flux: new tag ─► commit to main ─► Helm upgrade ─► http://atmos.lan
```

> k3s itself is installed by [project-atmos-onprem-ansible](https://github.com/Kernel-Club/project-atmos-onprem-ansible) (`roles/k3s/`). This repository holds only the workloads.

## Contents

| Path | What it does |
|---|---|
| `flux-system/` | Flux configuration, HelmRelease of the application, image automation |
| `charts/atmos/` | Helm chart: frontend, backend, MongoDB, Redis, Ingress, NetworkPolicy, memory limits |
| `monitoring/` | Prometheus, Grafana, kube-state-metrics (namespace `monitoring`) |

| Address | Service |
|---|---|
| `http://atmos.lan` | Application (`/` → frontend, `/api` → backend) |
| `http://grafana.atmos.lan` | Dashboards |

## Delivery

1. Jenkins pushes a new image to Nexus (`<build>-<short-sha>`).
2. Flux selects the highest build number and commits the tag to `spec.values` of the HelmRelease in `main`.
3. Flux upgrades the Helm release and the pods are replaced.

```bash
flux get helmreleases -A      # deployed version and failures
flux get images all -A        # scanned tags and selected image
```

**Rollback:**

```bash
flux suspend image update --all -n flux-system   # stop Flux from selecting the newest image again
git revert <tag-commit> && git push              # restore the previous version
flux resume image update --all -n flux-system    # after the fix is built
```

**Logs:** `kubectl logs -n atmos deploy/backend` (add `--previous` after a restart). There is no central log storage.

## Secrets

Not stored in Git. Created by hand with `kubectl create secret`: OpenWeather key, MongoDB password, Grafana admin password, Nexus pull credentials and CA for Flux. Flux uses a deploy key with write access to this repository only.

## Related

| Repository | Role |
|---|---|
| [project-atmos-dev](https://github.com/Kernel-Club/project-atmos-dev) | Application source and on-premise docs (`docs/on-premise/`) |
| [project-atmos-onprem-ansible](https://github.com/Kernel-Club/project-atmos-onprem-ansible) | Servers and k3s installation |
| [project-atmos-onprem-jenkins](https://github.com/Kernel-Club/project-atmos-onprem-jenkins) | CI pipeline that publishes the images |

Branching: `main` is the state of the cluster; changes via `feature/*` / `fix/*` and a Pull Request. Flux commits image tags directly to `main` ([branching model](https://github.com/Kernel-Club/project-atmos-dev/blob/develop/docs/on-premise/branching-model.md)).
