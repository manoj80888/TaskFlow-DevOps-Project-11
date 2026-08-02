# Architecture

## System Overview

```
Developer
    |
    v
GitHub Repository (source control)
    |
    v
GitHub Actions (CI/CD engine)
    |
    +--> Build & smoke-test Docker image
    |
    v
Docker Hub (image registry)
    |
    v
Kubernetes Cluster (Minikube)
    |
    +--> Deployment (3 replicas, self-healing, rolling updates)
    +--> Service (NodePort, load-balances across pods)
    +--> Ingress (host-based routing: taskflow.local)
    +--> HPA (auto-scales 2-10 pods based on CPU)
    |
    v
TaskFlow Application (NGINX serving static task-manager UI)
    |
    v
Prometheus (scrapes cluster + pod metrics)
    |
    v
Grafana (dashboards over Prometheus data)
```

## CI/CD Flow

1. Developer pushes to `main`.
2. GitHub Actions checks out the repo, validates static assets, builds the
   Docker image, runs a local smoke test (`curl` against a temporary
   container), then pushes the image to Docker Hub tagged with the
   GitHub Actions run number **and** `latest`.
3. The `deploy` job configures `kubectl` using the `KUBE_CONFIG` secret and
   runs `kubectl set image` to trigger a rolling update on the
   `taskflow-app` Deployment.
4. Kubernetes performs a rolling update: new pods come up (readiness probe
   must pass) before old pods are terminated, so there is no downtime.

## Monitoring Flow

- `kube-prometheus-stack` (Helm chart) installs Prometheus, Alertmanager,
  Grafana, node-exporter and kube-state-metrics into the `monitoring`
  namespace in one shot.
- Prometheus automatically scrapes cluster-level metrics (node CPU/memory,
  pod status, restarts) via node-exporter and kube-state-metrics — no
  extra config needed for infrastructure-level monitoring.
- Grafana is pre-wired to the Prometheus data source; community dashboards
  (IDs in `monitoring/grafana/dashboards.md`) visualize it immediately.
- App-level (custom) metrics are out of scope for a static NGINX site;
  `monitoring/prometheus/servicemonitor.yaml` is left as a template for if
  the app is later extended with an instrumented backend.

## Deployment Flow (environments)

`kubernetes/dev`, `kubernetes/test`, and `kubernetes/prod` hold
environment-specific Deployment/Service manifests (separate namespaces:
`taskflow-dev`, `taskflow-test`, `taskflow-prod`), while
`kubernetes/{configmap,secret,ingress,hpa}.yaml` are shared templates
applied against whichever namespace you're targeting. The default,
single-environment quickstart in this README uses the flat
`kubernetes/namespace.yaml` + `kubernetes/deployment.yaml` +
`kubernetes/service.yaml` (namespace `taskflow`).
