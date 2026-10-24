# Capstone 2: Robot Shop Observability & Reliability

This project focuses on deploying **Stan's Robot Shop** to Kubernetes and learning how to monitor its health, collect application metrics, and build the foundations for reliable operations.

## Environment and Tools

- **Amazon EKS** — managed Kubernetes cluster
- **Kubernetes and Helm** — deploy and manage the Robot Shop services
- **Prometheus** — scrape and query metrics
- **Grafana** — visualize metrics in dashboards
- **AWS EBS CSI / EBS volumes** — provide persistent storage for workloads that need it
- **Git and GitHub** — version control and project history

## What I Have Done So Far

### 1. Deployed Robot Shop to EKS

- Worked in my own fork of the Robot Shop repository.
- Used the project's Helm chart to deploy the microservices into the `robot-shop` namespace on EKS.
- Inspected Kubernetes deployments, pods, services, events, and scheduling issues while troubleshooting the deployment.
- Validated the Helm chart with `helm lint` and rendered its templates with `helm template`.

### 2. Set Up Prometheus and Grafana

- Deployed the Prometheus/Grafana monitoring stack with Helm in the `monitoring` namespace.
- Checked Prometheus target health and confirmed that the Catalogue service's metrics endpoint was being scraped successfully.
- Created a Grafana dashboard called **Robot Shop - Reliability Overview**.
- Added panels for desired replicas, available replicas, unavailable deployments, pod phases, and available replicas by deployment.
- Added/loaded Prometheus rules for detecting unavailable Robot Shop deployment replicas.

### 3. Instrumented the Catalogue Service

- Verified that the Catalogue service exposes application metrics, including `http_requests_total` and `http_request_duration_seconds`.
- Confirmed that the metrics endpoint is available at `/metrics` on port `8080`.
- Configured Prometheus discovery for the Catalogue `ServiceMonitor` using the monitoring release label.
- Verified that `serviceMonitor/robot-shop/catalogue/0` is **UP** in Prometheus.

## Evidence

### Grafana — Robot Shop Reliability Overview

The screenshot shows 12 desired replicas, 12 available replicas, and 0 unavailable deployments at the time it was captured. The dashboard also includes pod phase and per-deployment replica panels.

![Grafana Robot Shop Reliability Overview](docs/images/grafana-reliability-overview.png)
![Grafana Robot Shop Red Metrics](docs/images/grafana-red-metrics.png)

### Prometheus — Catalogue Metrics Target

The screenshot shows the Catalogue ServiceMonitor target scraping `http://<catalogue-pod-ip>:8080/metrics` successfully, with its state marked **UP**.

![Prometheus Catalogue target UP](docs/images/prometheus-catalogue-target.png)

## Knowledge Gained

- **Kubernetes health:** how to inspect pods, deployments, services, events, and scheduling failures to understand what is happening inside a cluster.
- **Helm:** how Helm charts package Kubernetes manifests, how to validate a chart with `helm lint`, and how to preview rendered resources with `helm template`.
- **Prometheus scraping:** Prometheus collects metrics from configured endpoints; a target marked `UP` means the scrape endpoint is reachable and responding successfully.
- **ServiceMonitor discovery:** Prometheus Operator uses `ServiceMonitor` resources and label selectors to discover which Kubernetes services should be scraped.
- **Application metrics:** counters such as `http_requests_total` can help measure request volume, while histograms such as `http_request_duration_seconds` can help analyze latency.
- **Grafana dashboards:** dashboards turn metric queries into visual signals that help operators quickly assess workload health.
- **Reliability signals:** desired versus available replicas and pod phases provide a starting point for detecting degraded deployments.
- **Troubleshooting:** storage provisioning, resource limits, pod capacity, and scheduling events can all affect whether workloads become healthy.

## Current Progress

**Completed so far:** Robot Shop deployment work, monitoring stack setup, the initial reliability dashboard, Catalogue metrics verification, and a successful Prometheus scrape of the Catalogue metrics endpoint.

**Still to complete:** centralized logging with Fluent Bit and Loki, distributed tracing with OpenTelemetry and Jaeger, a documented availability SLO and error-budget policy, Slack alerting with a linked runbook, and an incident drill with a blameless postmortem. The dashboard and monitoring configuration should also be exported and committed to the repository as code where applicable.

## Useful Commands

```bash
# Check Robot Shop workloads
kubectl get pods -n robot-shop
kubectl get deployments -n robot-shop
kubectl get services -n robot-shop

# Check monitoring workloads
kubectl get pods -n monitoring

# Validate and render the Helm chart
helm lint K8s/helm
helm template robot-shop K8s/helm --namespace robot-shop

# Inspect recent namespace events
kubectl get events -n robot-shop --sort-by='.metadata.creationTimestamp'
```

## Project Goal

The goal is not only to run Robot Shop, but to understand its health through metrics, logs, and traces, define measurable reliability objectives, alert on failures, and practise responding to incidents.
