# Dashboards and runbooks

## What is a dashboard?

A dashboard is a page containing charts, numbers, and links about a system. It turns telemetry into a quick answer to a question such as “Are users able to sign in?” or “Which dependency is making requests slow?” Each panel should help someone understand a problem or decide what to do next.

## Why do we need dashboards?

People cannot watch logs and metrics continuously. A dashboard brings important information together so an engineer can understand the current situation in a few minutes and gives the whole team the same view during an incident.

Dashboards help us see user impact, compare current behavior with a normal baseline, notice trends, check whether a deployment worked, and find the next place to investigate. For example, a checkout dashboard might show that errors increased after a deployment; a dependency panel can then show payment timeouts, with a link to a representative trace.

## Common dashboard types

### System or fleet dashboard

Shows the health of many services or hosts. Typical panels include total requests, availability, latency, active incidents, and regions with failures.

### Service dashboard

Shows one service and its user-facing endpoints: request rate, error rate, latency percentiles, SLO status, error budget, saturation, and recent deployments.

### Dependency dashboard

Shows databases, queues, caches, and external APIs: latency, errors, timeouts, connection pools, queue depth, and consumer lag.

### Business dashboard

Shows whether technical problems affect the product, such as sign-ups completed, orders accepted, payments declined, or messages delivered.

### Deployment dashboard

Compares a new release with the previous version using error rate, latency, resource usage, and key business outcomes.

## How to design one

Put user impact at the top, followed by likely causes. Use clear names, units, legends, and time ranges. Show a current value and a baseline. Annotate deployments, configuration changes, and incidents. Keep labels consistent so a metric, log, and trace can be connected.

Link important panels to their query, related logs, traces, and a runbook. Remove panels that no one uses. A dashboard that takes ten minutes to understand is not useful during a fast-moving outage.

## Common tools

| Tool | Common use |
| --- | --- |
| Grafana | Dashboards using Prometheus, Loki, Elasticsearch, SQL databases, and many other sources |
| Kibana | Explore Elasticsearch logs and create log and metric visualizations |
| Prometheus | Store and query time-series metrics; commonly paired with Grafana |
| Datadog | Hosted metrics, logs, traces, dashboards, and alerting |
| New Relic | Hosted application monitoring, dashboards, logs, and traces |
| Splunk | Log search, dashboards, security analysis, and machine data |
| Cloud provider tools | CloudWatch, Azure Monitor, and Google Cloud Monitoring for managed infrastructure |

Tool choice depends on existing infrastructure, query needs, retention, cost, and team skills. Good dashboard design matters more than the brand of the tool.

## Runbooks

A runbook is a short guide for responding to a known alert. It should explain the alert's meaning, user impact, first checks, safe mitigations, rollback criteria, escalation owner, and how to verify recovery. Include copyable queries and example log or trace fields. Test runbooks during drills; stale links and missing permissions are operational problems.
