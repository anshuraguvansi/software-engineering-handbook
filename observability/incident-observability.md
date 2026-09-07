# Observability during incidents

## What is an incident?

An incident is an unplanned problem that affects users, data, security, or an important internal service. Observability helps the team detect the problem, understand its impact, find a likely cause, restore service, and learn from what happened.

## Why do we need observability during an incident?

During an incident, information is incomplete and time matters. Logs, metrics, traces, dashboards, and profiles provide evidence instead of guesses. They help the team answer:

- What is broken?
- When did it start?
- Which users or regions are affected?
- Is the problem getting worse?
- What changed recently?
- Did the mitigation actually fix it?

## Who does what?

- **Incident commander:** coordinates the response, sets priorities, and keeps communication clear.
- **Investigator:** uses telemetry to form and test hypotheses.
- **Mitigator:** applies a safe rollback, configuration change, failover, or other recovery action.
- **Communicator:** updates stakeholders and support teams.

One person may fill several roles for a small incident, but the responsibilities should still be clear.

## A simple investigation process

1. **Confirm the alert.** Check the alert query, time window, and whether it represents real user impact.
2. **Measure the impact.** Look at the affected SLI, error rate, latency, regions, versions, and customer journeys.
3. **Find the start time.** Compare the affected period with a known-good period and check deployments, configuration changes, and dependency incidents.
4. **Follow one request.** Open a representative trace and inspect the slow or failed span.
5. **Add details.** Search logs using the trace ID, then inspect dependency metrics, queue depth, or a profile if resource usage is involved.
6. **Write down hypotheses.** Record the time, query, observation, and conclusion so people do not repeat the same work.
7. **Mitigate safely.** Prefer reversible actions such as rollback, disabling a feature flag, failing over, or reducing optional work.
8. **Verify recovery.** Confirm the user-facing SLI improves, check a fresh trace, and continue watching the alert's recovery window.

## Example: increased checkout latency

Suppose an alert reports that checkout is burning its error budget.

```text
Metric: p95 checkout latency increased from 400 ms to 3 s
Scope: only version 2026.09.07 in one region
Change: version 2026.09.07 was deployed 10 minutes ago
Trace: payment span takes 2.4 s; other spans are normal
Log: payment client reports connection-pool exhaustion
Action: roll back version 2026.09.07
Verification: latency returns to 400 ms and new traces complete normally
```

The metric detected the problem, the deployment comparison narrowed the scope, the trace identified the slow dependency, and the log explained the failure. The team can now investigate the connection-pool change after service is restored.

## After the incident

Preserve useful queries, dashboards, traces, and timestamps. Hold a blameless review focused on contributing conditions and improvements. Fix the detection gap, noisy alert, missing context, unsafe cardinality, or incomplete runbook that slowed the response. Add a regression check when practical, such as an alert-rule test, trace-propagation test, schema validation, synthetic journey, or Collector smoke test.

## Common mistakes

- Starting with infrastructure graphs before confirming user impact.
- Changing many things at once, making the cause impossible to identify.
- Treating one failed request as a complete outage without checking the overall rate.
- Closing the incident when the alert stops without verifying the user journey.
- Using production telemetry that contains secrets or personal data in chat messages.
