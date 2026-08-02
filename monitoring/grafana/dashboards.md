# Grafana Dashboards

Import these community dashboard IDs after logging into Grafana
(Dashboards → New → Import → enter ID):

| ID    | Name                        | Purpose                          |
|-------|-----------------------------|-----------------------------------|
| 315   | Kubernetes cluster monitoring (via Prometheus) | Node Exporter overview |
| 15757 | Kubernetes / Views / Global  | Cluster-wide health              |
| 1860  | Node Exporter Full           | Node CPU/Memory/Disk/Network     |

After importing, select the `Prometheus` data source that kube-prometheus-stack
already created for you (it's pre-wired — no manual data source setup needed).
