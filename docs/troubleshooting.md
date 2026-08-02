# Troubleshooting

## Image Pull Errors (`ErrImagePull` / `ImagePullBackOff`)

- Confirm the image name/tag in `kubernetes/deployment.yaml` matches what
  was actually pushed to Docker Hub (`docker.io/<username>/taskflow:<tag>`).
- Confirm the Docker Hub repo is **Public**, or add an
  `imagePullSecret` if private.
- `kubectl describe pod <pod-name> -n taskflow` shows the exact pull error.

## Pod CrashLoopBackOff

- `kubectl logs <pod-name> -n taskflow --previous` shows the last crash's logs.
- For this app (static NGINX), the most common cause is a broken
  `Dockerfile` COPY path — verify `index.html` and `styles.css` exist in
  `app/` and are spelled exactly as referenced.

## Ingress Not Working

- Confirm the addon is enabled: `minikube addons enable ingress`.
- Confirm `ingressClassName: nginx` is set (Kubernetes 1.19+ requires it).
- Add `taskflow.local` to your hosts file pointing at `minikube ip`:
  ```bash
  echo "$(minikube ip) taskflow.local" | sudo tee -a /etc/hosts
  ```

## Grafana Login Issues

```bash
kubectl get secret prometheus-grafana -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 -d
```
Default username is `admin` unless overridden in
`monitoring/prometheus/values.yaml`.

## Prometheus Not Scraping

- Check targets: port-forward Prometheus (`kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090 -n monitoring`) and open `http://localhost:9090/targets`.
- Infra-level metrics (CPU/memory/pod health) work out of the box via
  node-exporter and kube-state-metrics.
- App-level custom metrics require an instrumented app + ServiceMonitor
  (see `monitoring/prometheus/servicemonitor.yaml` template) — not enabled
  by default since the app is a static NGINX site with no `/metrics`
  endpoint.

## GitHub Actions `deploy` Job Fails / Hangs

- This almost always means the runner cannot reach your Kubernetes API
  server. GitHub-hosted runners cannot see a local Minikube cluster. See
  the "Important limitation" note in `docs/deployment-guide.md`.
