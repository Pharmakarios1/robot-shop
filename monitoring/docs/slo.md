# Service Level Objectives (SLOs)

## 1. Overview

A Service Level Objective (SLO) is a measurable reliability target for a service over a defined period.

For Stan's Robot Shop, SLOs help us evaluate whether the application is available, responds successfully, and provides acceptable performance.

These targets are proposed for the capstone and must be validated against actual application metrics.

## 2. Proposed SLOs

| Indicator | Proposed Objective | Measurement |
|---|---|---|
| Availability | 99.5% successful availability over 30 days | Successful health checks or valid service requests |
| Request Success Rate | At least 99.5% of eligible requests succeed over 30 days | Successful requests divided by eligible requests |
| Request Latency | 95% of eligible requests complete within 500 ms | HTTP request duration histogram |
| Monitoring Coverage | Critical application metrics are collected and visible | Prometheus targets and Grafana dashboards |

These are initial learning targets, not verified production guarantees.

## 3. Service Level Indicators (SLIs)

A Service Level Indicator (SLI) is the measurement used to evaluate an SLO.

### Availability SLI

Availability can be measured as:

Successful eligible requests / Total eligible requests × 100

### Request Success Rate SLI

The success rate measures the proportion of eligible requests that complete successfully.

For HTTP services, define which status codes count as successful. For example, HTTP 5xx responses normally indicate server errors, while 4xx responses may represent invalid client requests and should be handled according to the service's SLO definition.

### Latency SLI

Latency measures how long requests take to complete.

The proposed latency objective is:

95% of eligible requests complete within 500 milliseconds over the measurement window.

Use the application's request-duration histogram in Prometheus to evaluate this target. Confirm the histogram metric name and bucket boundaries before writing a production query.

## 4. Measurement Window

The proposed SLO measurement window is 30 days.

For a learning environment, shorter windows can help with debugging, but they should not be confused with the formal 30-day SLO window.

## 5. Monitoring

Prometheus collects application metrics, while Grafana visualizes them.

The existing RED dashboard provides useful signals:

- Request rate.
- HTTP 5xx error rate.
- 95th-percentile latency.
- Request volume by route.

These metrics support reliability analysis, but the SLOs cannot be considered verified until the relevant queries, measurement windows, and targets have been validated.

## 6. Review Checklist

- [ ] Define the eligible requests for each SLI.
- [ ] Confirm the available Prometheus metric names and labels.
- [ ] Create queries that measure each SLI over the selected window.
- [ ] Compare the measured values with the proposed targets.
- [ ] Record results and evidence in the capstone README.
- [ ] Review the targets after collecting sufficient data.

## 7. Important Note

An SLO is a target, not proof that the target has been met.

Report measured compliance only when supported by collected metrics over the specified measurement window.
