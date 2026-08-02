# Deployment Guide

## Prerequisites

| Tool | Purpose | Install |
|---|---|---|
| Git | version control | https://git-scm.com/downloads |
| Docker Desktop / Engine | build & run containers | https://docs.docker.com/get-docker/ |
| A Docker Hub account | image registry | https://hub.docker.com/signup |
| A GitHub account | source + CI/CD | https://github.com/signup |
| kubectl | talk to Kubernetes | https://kubernetes.io/docs/tasks/tools/ |
| Minikube | local Kubernetes cluster | https://minikube.sigs.k8s.io/docs/start/ |
| Helm 3 | install Prometheus/Grafana | https://helm.sh/docs/intro/install/ |

## 1. Local Docker Setup

```bash
docker build -t taskflow:v1 ./app
docker run -d -p 8080:80 --name taskflow taskflow:v1
```

Open http://localhost:8080 — you should see the TaskFlow app with
"Add Task" working (it's client-side only for now).

## 2. Kubernetes Setup

```bash
minikube start
minikube addons enable ingress
kubectl apply -f kubernetes/namespace.yaml
kubectl apply -f kubernetes/configmap.yaml
kubectl apply -f kubernetes/secret.yaml
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
kubectl apply -f kubernetes/ingress.yaml
kubectl apply -f kubernetes/hpa.yaml
```

## 3. Deployment Steps (CI/CD)

1. Push code to `main` on GitHub.
2. GitHub Actions builds, tests, and pushes the image to Docker Hub.
3. GitHub Actions runs `kubectl set image` against your cluster.

**Important limitation:** GitHub-hosted runners live in GitHub's cloud and
cannot reach a Minikube cluster running on your laptop. The `deploy` job in
`.github/workflows/pipeline.yml` will only succeed against a
publicly-reachable cluster (e.g. a managed cluster, or a self-hosted GitHub
Actions runner installed *on the same machine* as Minikube). For a
pure-local learning setup, either:

- Run a **self-hosted runner** on your machine (Settings → Actions →
  Runners → New self-hosted runner), or
- Treat CI (build/test/push to Docker Hub) as fully automated, and do the
  `kubectl apply` / `kubectl set image` step manually or via a local script
  for the CD half.

## 4. Validation

```bash
kubectl get pods -n taskflow
kubectl get svc -n taskflow
minikube service taskflow-service -n taskflow --url
kubectl rollout status deployment/taskflow-app -n taskflow
```

## 5. Secrets Setup (GitHub)

Repository → Settings → Secrets and variables → Actions → New repository secret:

| Name | Value |
|---|---|
| `DOCKER_USERNAME` | your Docker Hub username |
| `DOCKER_TOKEN` | Docker Hub → Account Settings → Personal Access Tokens |
| `KUBE_CONFIG` | output of `cat ~/.kube/config` (only if using a self-hosted runner / reachable cluster) |
