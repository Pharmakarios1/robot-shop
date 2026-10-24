# Observability Implementation Status

## Project

Stan's Robot Shop – Capstone 2

## Current Components

| Component | Purpose | Status |
|---|---|---|
| Prometheus | Metrics collection | Previously deployed |
| Grafana | Metrics dashboards and visualization | Previously deployed |
| Catalogue ServiceMonitor | Scrapes catalogue metrics | Previously observed as UP |
| Loki | Log storage and querying | Pod running; end-to-end logging not verified |
| Fluent Bit | Collects container logs and forwards them to Loki | Deployment unhealthy; requires troubleshooting |
| Grafana Tempo | Distributed trace storage | Configuration prepared; deployment not verified |
| OpenTelemetry Collector | Receives and forwards traces | Not yet verified |
| Robot Shop tracing instrumentation | Generates distributed traces | Not yet verified |

Statuses describe the latest recorded project evidence and must be checked again before presenting them as current.

## Metrics

The project has a Grafana RED metrics dashboard covering:

- Request rate.
- HTTP 5xx error rate.
- 95th-percentile request latency.
- Request volume by route.

A Prometheus Targets screenshot previously showed the catalogue ServiceMonitor as UP. Preserve this evidence in the project README.

## Logging

Loki was running, but the Fluent Bit deployment had Pending pods and a CrashLoopBackOff pod.

Therefore, the logging pipeline must not be described as fully operational until Fluent Bit is healthy and logs can be queried successfully in Grafana.

See the planned logging documentation and configuration under `monitoring/logging/`, where available.

## Tracing

The proposed tracing pipeline is:

Robot Shop -> OpenTelemetry Collector -> Grafana Tempo -> Grafana

The Tempo configuration has been prepared, but chart compatibility must be validated before deployment. Application instrumentation and trace delivery also remain to be verified.

See `monitoring/docs/tracing.md`.

## Resource Considerations

The project runs on a limited EKS environment. Additional monitoring components consume CPU, memory, and potentially persistent storage.

Before deploying additional services:

1. Inspect node capacity and pod resource usage.
2. Check for Pending and CrashLoopBackOff pods.
3. Avoid scaling the cluster or adding components without reviewing the resource and cost impact.
4. Prefer validating Helm manifests before installing new releases.

## Completion Criteria

- [ ] Metrics dashboards and Prometheus targets verified.
- [ ] Fluent Bit healthy and logs queryable in Loki.
- [ ] Tempo deployed successfully.
- [ ] OpenTelemetry Collector healthy.
- [ ] Robot Shop generates distributed traces.
- [ ] Grafana can retrieve and display a real trace.
- [ ] Screenshots and final results added to the main README.

## Evidence Policy

Document only what has been tested. Configuration files and architecture diagrams describe intended behavior; they are not proof of successful deployment.
