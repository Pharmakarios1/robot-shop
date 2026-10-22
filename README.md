# Robot Shop Observability & Reliability Engineering on AWS EKS

## Overview

This project explores how to deploy, monitor, troubleshoot, and improve the reliability of a microservices application running on Amazon Elastic Kubernetes Service (EKS).

The project uses **Robot Shop** as the application workload and combines Kubernetes, Helm, Prometheus, Grafana, Fluent Bit, Loki, OpenTelemetry, and Jaeger to build an observability and reliability engineering solution.

The objective goes beyond deploying an application. It is to understand how to detect service degradation, investigate failures, trace requests across microservices, define measurable reliability targets, manage error budgets, and respond to incidents systematically.

## Project Objectives

- Deploy Robot Shop to Amazon EKS using Kubernetes and Helm.
- Generate application traffic and observe system behavior under load.
- Collect and visualize application and infrastructure metrics.
- Centralize container logs and make them searchable.
- Implement distributed tracing for requests passing through microservices.
- Define Service Level Objectives (SLOs) and understand error-budget policies.
- Configure actionable alerts for reliability issues.
- Conduct controlled incident drills and document findings in blameless postmortems.

## Architecture

The intended observability pipeline is:

```text
                     Users / Load Generator
                              |
                              v
                       Robot Shop App
                    (Microservices on EKS)
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
        Prometheus        Fluent Bit      OpenTelemetry
        Metrics           Log Collection   Instrumentation
             |                |                |
             v                v                v
          Grafana            Loki            Jaeger
       Visualization      Log Storage     Trace Analysis
             |
             v
       Dashboards & Alerts
             |
             v
       Incident Response
       SLOs & Error Budgets
```

This diagram represents the intended architecture. Individual components must be validated end to end before the complete pipeline can be considered operational.

## Technology Stack

| Technology | Purpose |
|---|---|
| AWS EKS | Managed Kubernetes control plane for running the application |
| Kubernetes | Container orchestration, service discovery, scheduling and workload management |
| Helm | Packaging, configuring, installing and upgrading Kubernetes applications |
| Robot Shop | Microservices application used as the workload |
| Prometheus | Collecting and querying time-series metrics |
| Grafana | Dashboards, metric exploration, log exploration and alert visualization |
| Fluent Bit | Lightweight collection, processing and forwarding of container logs |
| Loki | Centralized log storage and LogQL-based log queries |
| OpenTelemetry | Instrumenting applications and collecting telemetry, including distributed traces |
| Jaeger | Exploring distributed traces and analyzing request paths and latency |
| Load generator | Producing traffic to observe system behavior under load |

---

## 1. Kubernetes and Amazon EKS

Amazon EKS provides a managed Kubernetes control plane, while worker nodes run application and observability workloads.

Kubernetes is responsible for maintaining the desired state of workloads, scheduling pods, managing service discovery, and restarting failed containers when appropriate.

### Key concepts learned

**Pods:** The smallest deployable Kubernetes units. A pod contains one or more containers.

**Deployments:** Manage replicated, stateless application workloads and support rolling updates.

**Services:** Provide stable network endpoints for communicating with application pods.

**Namespaces:** Organize resources into logical groups, such as `robot-shop` and `monitoring`.

**DaemonSets:** Ensure that a pod runs on each eligible node. This is useful for node-level agents such as Fluent Bit.

**Resource requests and limits:** Requests influence scheduling, while limits constrain resource consumption.

**Node affinity, taints and tolerations:** Control which nodes are suitable for particular workloads.

### Operational troubleshooting

A pod can remain Pending even when a cluster has available nodes. The scheduler evaluates resource capacity, node eligibility, affinity rules, taints and other constraints.

Useful diagnostic commands include:

```bash
kubectl get nodes
kubectl get pods -A
kubectl describe pod <pod-name> -n <namespace>
kubectl get events -A --sort-by=.lastTimestamp
kubectl describe node <node-name>
```

The key operational lesson is to diagnose scheduling events before modifying workloads or adding infrastructure.

## 2. Helm and Kubernetes Release Management

Helm packages Kubernetes manifests into charts and allows configuration through values files.

This makes it easier to install and maintain complex applications such as Robot Shop, Prometheus, Grafana and Loki.

### Key concepts learned

- **Chart:** A package containing Kubernetes templates and related metadata.
- **Values file:** Configuration used to customize a chart.
- **Release:** An installed instance of a Helm chart.
- **Upgrade:** Applies changes to an existing release.
- **Rollback:** Restores a previous release revision when an appropriate rollback is available.
- **Template rendering:** Generates Kubernetes manifests without deploying them.
- **Linting:** Checks a local chart for common structural and configuration issues.

Example commands:

```bash
helm list -A
helm history <release-name> -n <namespace>
helm get values <release-name> -n <namespace>
helm template <release-name> <chart> -f values.yaml
helm upgrade --install <release-name> <chart> \
  -n <namespace> -f values.yaml
```

A successful template render does not guarantee that the resulting workloads will schedule or become healthy. Deployment status, pod events and application logs must also be examined.

A Helm command that times out may leave resources created in the cluster even though the release is marked failed. Inspecting the actual Kubernetes state is therefore essential.

## 3. Metrics with Prometheus

Prometheus collects numerical measurements over time and stores them as time-series data. These measurements help determine whether an application is healthy and how its behavior changes under load.

### Important metric categories

**Request rate:** How many requests the application receives over a period.

**Error rate:** The proportion or number of requests that fail.

**Latency:** How long requests take to complete, commonly examined through percentiles such as p50, p95 and p99.

**Resource utilization:** CPU, memory and other infrastructure measurements.

**Availability:** Whether a service successfully responds to qualifying requests according to the chosen measurement method.

### Example PromQL queries

If the application exposes compatible metrics, request rate can be calculated using:

```promql
sum(rate(http_requests_total[5m]))
```

The request error ratio can be calculated with a suitable HTTP request counter:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

These examples require an application metric named `http_requests_total` with a suitable `status` label. Actual metric names and labels must be verified against the application's exposed metrics.

Metrics that are not exposed or collected cannot be queried reliably. Kubernetes infrastructure metrics and application-level request metrics are different, and both may be needed for a complete view.

## 4. Visualization and Exploration with Grafana

Grafana provides a unified interface for exploring and visualizing observability data.

It can use Prometheus for metrics and Loki for logs. With distributed tracing configured, it can also support trace exploration and links between telemetry sources.

### Key concepts learned

- **Data sources:** Connections to systems such as Prometheus and Loki.
- **Dashboards:** Collections of visualizations organized around operational questions.
- **Panels:** Individual charts, tables, or other visual representations.
- **Explore:** An interactive interface for querying telemetry without first building a dashboard.
- **Alerting:** Evaluating conditions and notifying responders when defined rules are met.

Useful dashboards should answer questions such as:

- Is the application receiving traffic?
- Are errors increasing?
- Is request latency degrading?
- Which service is consuming excessive resources?
- Did the incident begin after a deployment?
- What do the application logs report during the failure?

A dashboard is most useful when its panels are tied to concrete operational decisions rather than simply displaying available metrics.

## 5. Centralized Logging with Fluent Bit and Loki

Logs provide event-level detail about application behavior. They can reveal errors, exceptions, connection failures and other events that aggregate metrics cannot explain on their own.

### Fluent Bit

Fluent Bit is a lightweight log processor and forwarder. It can collect container logs from worker nodes, enrich records with Kubernetes metadata, process records and send them to a destination.

A DaemonSet is a common deployment pattern because it allows the collector to run on each eligible worker node.

### Loki

Loki stores and queries logs using labels and LogQL. Unlike a conventional database query against every log field, Loki primarily indexes stream labels and searches the corresponding log content.

The intended pipeline is:

```text
Robot Shop containers
        |
        v
Kubernetes container log files
        |
        v
Fluent Bit DaemonSet
        |
        v
Loki HTTP ingestion endpoint
        |
        v
Grafana Explore
```

### Example LogQL queries

Find logs associated with the Robot Shop namespace:

```logql
{namespace="robot-shop"}
```

Search for messages containing an error:

```logql
{namespace="robot-shop"} |= "error"
```

Filter for a specific service or pod using the labels actually present in Loki:

```logql
{namespace="robot-shop", container="shipping"}
```

LogQL queries depend on the labels that Fluent Bit successfully attaches to the records. Label names and values should be verified in Grafana Explore before relying on a query.

### Logging operational lessons

- A running Loki pod does not prove that logs are arriving.
- A running Fluent Bit pod does not prove that log delivery is successful.
- Output plugin availability, configuration syntax, network connectivity and ingestion responses must be checked.
- DaemonSet pods may remain Pending because of node capacity, affinity, taints or other scheduling constraints.
- Log retention, storage capacity and label cardinality should be considered when designing a production logging system.

For this lab, Loki is configured as a single-binary deployment with a persistent `gp3` volume. This is a practical learning configuration, not a claim that the deployment provides production-grade high availability.

## 6. Distributed Tracing with OpenTelemetry and Jaeger

Metrics show aggregate behavior, and logs provide detailed events. Distributed traces show how an individual request travels through a distributed application.

### OpenTelemetry

OpenTelemetry is a vendor-neutral observability framework for instrumenting applications and generating or exporting telemetry. It supports traces, metrics and logs.

For distributed tracing, an application creates spans representing operations. Related spans are connected through trace context so that operations belonging to the same request can be correlated.

### Jaeger

Jaeger provides trace storage and visualization. It helps engineers inspect the sequence of operations involved in a request, compare span durations and identify where latency or errors occur.

### Trace concepts

**Trace:** The end-to-end record of an operation across one or more services.

**Span:** A timed operation within a trace.

**Trace context propagation:** Passing trace identifiers between services so that spans can be correlated.

**Parent-child relationship:** Represents the relationship between operations within a trace.

**Instrumentation:** Adding telemetry generation to an application or library.

### Intended tracing workflow

1. Select a Robot Shop service that can be instrumented.
2. Configure OpenTelemetry instrumentation or an appropriate SDK.
3. Export traces to the chosen tracing backend or collector.
4. Configure Jaeger to receive or access the exported traces.
5. Generate application traffic.
6. Inspect traces to understand request paths, duration and errors.
7. Correlate trace findings with relevant Prometheus metrics and Loki logs where possible.

The purpose is to investigate questions such as: Which service contributes most to request latency? Does a downstream dependency fail? Does the failure occur consistently for one request path?

OpenTelemetry instrumentation and trace export must be implemented and verified before the tracing portion of the project can be considered complete.

## 7. Service Level Objectives (SLOs)

An SLO is a measurable reliability target for a service over a defined period. It provides a way to evaluate reliability from the user's perspective rather than relying solely on whether pods are running.

An SLO is typically built around a Service Level Indicator (SLI).

- **SLI:** The measurement used to evaluate service performance.
- **SLO:** The target for that measurement.
- **SLA:** An agreement that may specify service commitments and consequences when those commitments are not met.

### Example availability SLO

Suppose the project adopts this illustrative target:

**99.5% successful requests over a rolling 30-day window.**

The SLI could be calculated as:

```text
Availability SLI =
Successful eligible requests
---------------------------
Total eligible requests
```

The precise definition of a successful request must be documented. For example, the project may classify qualifying HTTP responses as successful and exclude health-check traffic.

For a 99.5% target:

```text
Allowed failure fraction = 1 - 0.995 = 0.005
```

This means that up to 0.5% of eligible requests may fail during the measurement window while the SLO remains satisfied.

An availability SLO must be based on trustworthy measurements. Kubernetes pod readiness alone does not prove that users can successfully complete their requests.

## 8. Error Budgets and Reliability Policy

An error budget is the amount of unreliability permitted by an SLO during its measurement window.

For a 99.5% availability SLO, the error budget is 0.5% of eligible requests.

If 100,000 requests qualify for the measurement window, the illustrative budget is:

```text
100,000 × 0.005 = 500 failed requests
```

If 400 of those requests fail, 80% of the request-based error budget has been consumed.

```text
Error budget consumption =
Observed failures
-----------------
Allowed failures
```

This calculation is meaningful only when the observed failures and allowed failures use the same SLI definition and measurement window.

### Error-budget policy

A reliability policy defines what the team does when the budget is being consumed.

An example policy could be:

- **Healthy budget:** Continue planned releases while monitoring service health.
- **Elevated consumption:** Investigate error spikes, latency degradation and risky changes.
- **Budget nearly exhausted:** Prioritize reliability improvements and review high-risk releases.
- **Budget exhausted:** Consider pausing nonessential risky releases until reliability improves, according to the team's agreed policy.

An error budget is not permission to cause failures. It is a framework for balancing delivery speed with reliability.

The project should calculate budget consumption from actual SLI measurements rather than treating the illustrative numbers above as observed project results.

## 9. Alerting and Incident Detection

Alerting turns telemetry into an operational signal requiring investigation or action.

Useful alerts should be based on symptoms that affect users or indicate a meaningful risk to reliability.

Examples include:

- High request failure rates.
- Sustained latency above a defined threshold.
- An application becoming unavailable.
- Excessive resource pressure.
- Rapid error-budget consumption.
- A logging or telemetry pipeline becoming unavailable.

### Alert design principles

1. Define the condition and its duration.
2. Identify the metric or signal that supports the condition.
3. Set an appropriate severity.
4. Provide enough context for diagnosis.
5. Route the alert to the appropriate responder.
6. Test that the alert fires and that notifications are delivered.
7. Review false positives and missed incidents.

Slack can be used as a notification destination, provided the integration is configured and tested.

An alert rule is not complete merely because it exists in configuration. It should be tested end to end, including its evaluation, notification routing and recovery behavior.

## 10. Incident Drills and Blameless Postmortems

An incident drill is a controlled exercise used to evaluate how well the system and its operators detect, investigate and recover from a failure.

The purpose is to practice the incident response process without waiting for a real production outage.

### Example incident drill

A controlled exercise could introduce a temporary failure in a selected Robot Shop dependency in a non-production environment.

The exercise should follow this sequence:

1. **Establish a baseline:** Record normal request rates, error rates and latency.
2. **Introduce a controlled fault:** Apply a documented and reversible failure condition.
3. **Detect:** Observe whether dashboards or alerts reveal the impact.
4. **Investigate:** Use metrics to identify the affected behavior, logs to inspect errors, and traces to locate the failing operation where tracing is available.
5. **Mitigate:** Restore the dependency or reverse the injected fault.
6. **Verify recovery:** Confirm that requests succeed and telemetry returns to the expected state.
7. **Document:** Record the timeline, impact, evidence, root cause and corrective actions.

### Blameless postmortem structure

A useful postmortem includes:

- **Summary:** What happened?
- **Impact:** Which users or services were affected?
- **Timeline:** When did the incident begin, get detected, and get resolved?
- **Detection:** Which metric, log, trace or alert revealed the issue?
- **Root cause:** What technical conditions caused the failure?
- **Contributing factors:** Why was the failure possible, or why was recovery delayed?
- **Response:** What actions mitigated the impact?
- **Corrective actions:** What changes will reduce the likelihood or impact of recurrence?
- **Owners and deadlines:** Who will complete each follow-up action?

A blameless approach focuses on system conditions, design decisions, observability gaps and process improvements rather than assigning personal blame.

Incident drills should be reversible, scoped to the lab and designed to avoid unnecessary disruption to shared infrastructure.

## 11. Load Generation and Performance Analysis

Load generation helps reveal application behavior under traffic.

A load generator can expose issues that are not visible when the application receives little or no traffic, such as rising latency, resource contention, dependency bottlenecks or increasing error rates.

A useful test process is:

1. Record baseline application and infrastructure metrics.
2. Generate a known workload.
3. Observe request rates, latency and failures.
4. Inspect resource utilization and dependency behavior.
5. Compare the results with the baseline.
6. Determine whether the application remains within its reliability targets.

A load test is useful only when the traffic generated and the measurements collected are understood. The existence of a load-generator pod alone does not prove that meaningful traffic is being produced.

## 12. Troubleshooting Methodology

A key learning outcome from this project is the ability to diagnose problems systematically.

The general troubleshooting cycle is:

1. Understand the component's expected responsibility.
2. Establish what healthy behavior should look like.
3. Inspect actual resource states.
4. Read Kubernetes events and application logs.
5. Form a hypothesis supported by evidence.
6. Apply the smallest appropriate change.
7. Verify the result with observable evidence.
8. Document the root cause and lessons learned.

Useful commands include:

```bash
# Inspect Kubernetes resources
kubectl get pods -A
kubectl get deployments,daemonsets -A

# Inspect scheduling and container failures
kubectl describe pod <pod-name> -n <namespace>
kubectl get events -A --sort-by=.lastTimestamp

# Inspect container logs
kubectl logs <pod-name> -n <namespace> --tail=100

# Inspect Helm releases
helm list -A
helm history <release-name> -n <namespace>

# Inspect Prometheus and Loki workloads
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

A configuration can be syntactically valid but operationally incorrect. Successful troubleshooting requires verifying both configuration and runtime behavior.

## 13. Current Implementation and Validation Status

The project should distinguish between deployed components, partially validated integrations and planned work.

| Area | Status |
|---|---|
| AWS EKS cluster and worker nodes | Deployed |
| Robot Shop microservices | Running; application health should continue to be monitored |
| Helm-based application management | In use |
| Prometheus and Grafana | Deployed; metrics have been explored |
| Loki single-binary deployment | Deployed; readiness endpoint responds and persistent volume is bound |
| Fluent Bit DaemonSet | Partially running; scheduling constraints require investigation |
| End-to-end log ingestion into Loki | Requires verification |
| Grafana Loki data source and log queries | Requires validation |
| OpenTelemetry instrumentation and Jaeger tracing | Planned implementation and validation |
| Defined SLI/SLO with measured error budget | Requires implementation and measurement |
| Tested reliability alerts and Slack notification | Requires implementation and end-to-end testing |
| Controlled incident drill and postmortem | Requires execution and documentation |

This status table reflects the observed project state and should be updated as each milestone is completed.

## 14. Roadmap and Next Steps

### Phase 1 — Complete logging

- Diagnose why Fluent Bit pods are Pending.
- Ensure Fluent Bit runs on eligible worker nodes.
- Verify that the Loki output plugin initializes correctly.
- Confirm that logs reach Loki.
- Add and test Loki as a Grafana data source.
- Create useful LogQL queries for Robot Shop services.

### Phase 2 — Improve metrics and dashboards

- Identify available application-level request metrics.
- Build dashboards for request rate, error rate and latency where instrumentation supports them.
- Add infrastructure resource panels.
- Generate load and record the observed behavior.

### Phase 3 — Implement distributed tracing

- Select a suitable Robot Shop service.
- Configure OpenTelemetry instrumentation.
- Configure trace export and Jaeger.
- Generate requests and inspect traces.
- Correlate traces with relevant metrics and logs.

### Phase 4 — Establish reliability targets

- Define a measurable availability SLI.
- Set an explicit SLO and measurement window.
- Calculate the error budget from actual measurements.
- Define a documented policy for elevated budget consumption.
- Test alerts against meaningful reliability conditions.
- Configure and verify Slack notifications.

### Phase 5 — Conduct an incident drill

- Select a controlled failure scenario.
- Establish a baseline.
- Inject and reverse the fault safely.
- Measure detection and recovery.
- Investigate using available telemetry.
- Write a blameless postmortem.
- Track corrective actions.

## 15. Key Learning Outcomes

This project develops practical understanding of:

- Kubernetes workload management, scheduling and resource constraints.
- Helm-based configuration and release troubleshooting.
- Metrics collection, querying and visualization.
- Centralized container logging and log-query design.
- Distributed tracing and trace-context propagation.
- Reliability measurement using SLIs and SLOs.
- Error-budget calculations and reliability policies.
- Alert design and notification validation.
- Controlled incident response and blameless postmortems.
- Evidence-driven troubleshooting across multiple observability systems.

The central lesson is that **deploying observability tools is only the beginning**. A reliable observability solution must collect accurate telemetry, connect that telemetry to operational questions, detect meaningful failures, support investigation and help the team demonstrate recovery.

## Conclusion

The Robot Shop observability capstone combines cloud infrastructure, Kubernetes operations and reliability engineering into a practical learning environment.

By completing the remaining logging, tracing, SLO, alerting and incident-response milestones, the project will demonstrate not only the ability to deploy infrastructure and monitoring tools, but also the ability to use evidence to understand system behavior, respond to failures and improve reliability over time.
