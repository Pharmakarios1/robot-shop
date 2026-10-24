# Distributed Tracing – Stan's Robot Shop

## Overview
Distributed tracing follows a request through application components and helps identify latency and failures.

## Proposed architecture

```text
User Request
     |
     v
Instrumented Robot Shop services
     | OTLP
     v
OpenTelemetry Collector
     |
     v
Grafana Tempo
     |
     v
Grafana
```

This is a proposed architecture, not a verified running pipeline. Robot Shop must generate trace data; installing Tempo alone does not instrument the application.

## Configuration
The proposed Tempo values are in `monitoring/tracing/tempo-values.yaml`. They were intended to configure a single replica, OTLP gRPC on port 4317, OTLP HTTP on port 4318, the query endpoint on port 3200, and a 5 GiB `gp3` persistent volume. Validate the values against a supported chart before deploying: an earlier `grafana/tempo` chart render produced a deprecation warning.

## Deployment and verification plan
1. Verify the supported Helm chart and its values schema.
2. Render templates and check the exit code before installing.
3. Check EKS capacity and existing Pending or CrashLoopBackOff pods.
4. Deploy Tempo only when capacity allows; verify pods, service, and PVC.
5. Configure an OpenTelemetry Collector receiver, processor, exporter, and service pipeline.
6. Instrument relevant Robot Shop services and send traces to the collector.
7. Configure Tempo as a Grafana data source and find a real Robot Shop trace.

## Verification checklist
- [ ] Supported chart renders successfully.
- [ ] Tempo pods are running and persistent storage is bound.
- [ ] OpenTelemetry Collector is healthy and exports to Tempo.
- [ ] Robot Shop generates trace data.
- [ ] Grafana can retrieve and display a real trace.

## Current status
**Planned; end-to-end tracing has not been verified.** The collector, application instrumentation, Grafana data source, and trace queries still require implementation and validation.

## References
- https://opentelemetry.io/docs/
- https://grafana.com/docs/tempo/latest/
- https://grafana.com/docs/grafana/latest/datasources/
