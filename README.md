# TaskFlow-DevOps

**Automated Build, Test, Containerization, Deployment and Monitoring Platform**

An end-to-end DevOps project demonstrating a real CI/CD pipeline: a
developer pushes code, GitHub Actions builds/tests/pushes a Docker image,
Kubernetes deploys it with rolling updates, self-healing, and
autoscaling, and Prometheus + Grafana monitor the whole thing.

```
Developer -> GitHub -> GitHub Actions -> Docker Build -> Docker Hub
   -> Kubernetes -> TaskFlow App -> Prometheus -> Grafana
```

## Stack

| Layer | Tool | Cost |
|---|---|---|
| Source control | GitHub | Free |
| CI/CD | GitHub Actions | Free |
| Containerization | Docker | Free |
| Registry | Docker Hub | Free |
| Orchestration | Kubernetes (Minikube) | Free |
| Monitoring | Prometheus | Free |
| Dashboards | Grafana | Free |

## Repository Structure

```
TaskFlow-DevOps/
├── .github/workflows/pipeline.yml   # CI/CD pipeline
├── app/                              # Task Management web app + Dockerfile
├── kubernetes/                       # Base manifests + dev/test/prod overlays
├── monitoring/                       # Prometheus & Grafana config/reference
├── docs/                             # Architecture, deploy guide, runbook, troubleshooting
├── screenshots/                      # Evidence for your portfolio/resume
└── README.md
```

## Quickstart

```bash
# 1. Run locally with Docker
docker build -t taskflow:v1 ./app
docker run -d -p 8080:80 --name taskflow taskflow:v1
open http://localhost:8080   # (or just visit it in your browser)

# 2. Deploy to Kubernetes
minikube start
minikube addons enable ingress
kubectl apply -f kubernetes/namespace.yaml
kubectl apply -f kubernetes/configmap.yaml
kubectl apply -f kubernetes/secret.yaml
kubectl apply -f kubernetes/deployment.yaml
kubectl apply -f kubernetes/service.yaml
kubectl apply -f kubernetes/hpa.yaml
minikube service taskflow-service -n taskflow --url

# 3. Add monitoring
kubectl create namespace monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack \
  -n monitoring -f monitoring/prometheus/values.yaml
kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring
```

Full step-by-step instructions: [`docs/deployment-guide.md`](docs/deployment-guide.md).

## Features Demonstrated

Git & GitHub · GitHub Actions CI/CD · Docker · Docker Hub · Kubernetes
Deployments/Services/Ingress · ConfigMaps & Secrets · Horizontal Pod
Autoscaling · Rolling Updates & Rollbacks · Self-Healing · Prometheus ·
Grafana · DEV/TEST/PROD environment separation

## Docs

- [Architecture](docs/architecture.md)
- [Deployment Guide](docs/deployment-guide.md)
- [Runbook](docs/runbook.md)
- [Troubleshooting](docs/troubleshooting.md)

## Resume Line

`TaskFlow-DevOps | End-to-End CI/CD and Kubernetes Platform`

## License

MIT — see [LICENSE](LICENSE).
