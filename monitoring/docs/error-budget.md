# Error Budget

## 1. Overview

An error budget is the amount of unreliability allowed by an SLO during its measurement window.

It helps us balance reliability with the need to release changes and improve the application.

For this capstone, the error budget is calculated from the proposed 99.5% availability SLO over 30 days.

## 2. Proposed Reliability Target

- SLO: 99.5% availability.
- Measurement window: 30 days.
- Allowed unreliability: 0.5%.

The error budget is derived from the selected SLO. It is not a separate reliability target.

## 3. Error Budget Calculation

Error budget percentage:

100% - SLO target percentage

For a 99.5% SLO:

100% - 99.5% = 0.5%

Over a 30-day period:

30 days × 24 hours × 60 minutes = 43,200 minutes

Allowed downtime equivalent:

43,200 × 0.005 = 216 minutes

Therefore, a 99.5% availability target over 30 days corresponds to approximately 216 minutes, or 3 hours and 36 minutes, of downtime.

This time-based calculation is illustrative. If the SLO is request-based, calculate the error budget from eligible requests and failed requests rather than assuming all minutes have equal traffic.

## 4. Request-Based Error Budget

For a request-success SLO, calculate:

Allowed failed requests = Total eligible requests × Allowed failure percentage

For a 99.5% request-success target, the allowed failure percentage is 0.5%.

Example:

If the service receives 100,000 eligible requests during the measurement window:

100,000 × 0.005 = 500 allowed failed requests

This is an example only; use actual request counts when evaluating the Robot Shop application.

## 5. Error Budget Consumption

Error budget consumption describes how much of the allowed budget has been used.

Budget consumption percentage:

Failed requests / Allowed failed requests × 100

For a time-based availability budget, the equivalent calculation is:

Observed downtime / Allowed downtime × 100

Interpretation:

- Below 100%: The budget has not been fully consumed.
- At 100%: The budget has been exhausted.
- Above 100%: The observed failures exceed the SLO allowance.

Use the calculation that matches the SLI defined in `slo.md`. Do not mix request-based and time-based budgets.

## 6. Operational Response

The following is a proposed response policy for this learning project:

### Budget Healthy

When consumption is low, continue normal development while monitoring reliability.

### Budget Depleting Quickly

Investigate error-rate increases, latency changes, failed deployments, and resource pressure.

### Budget Exhausted

Prioritize restoring reliability, investigate the cause, and defer risky changes until the service is stable.

These are suggested operational practices, not automated enforcement rules currently implemented in the cluster.

## 7. Prometheus and Grafana

Use the existing Grafana RED dashboard to investigate:

- HTTP 5xx error rates.
- Request volume.
- Request latency.
- Changes following deployments.

Before calculating the actual error budget, confirm that Prometheus has the required request counters and that their labels and semantics match the chosen SLI.

## 8. Review Checklist

- [ ] Confirm the SLO and its measurement window.
- [ ] Define the eligible requests or availability measurement.
- [ ] Calculate the allowed failure budget.
- [ ] Measure observed failures using Prometheus.
- [ ] Calculate budget consumption.
- [ ] Investigate rapid consumption or exhaustion.
- [ ] Document the findings and supporting Grafana evidence.

## 9. Important Note

The values in this document are proposed learning targets and examples. No claim is made that the Robot Shop application currently meets its SLO or has a particular amount of remaining error budget.

Actual results require valid measurements over the defined window.
