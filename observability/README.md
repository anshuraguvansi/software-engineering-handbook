# Observability

Observability is the ability to explain a system's internal state from the signals it emits. Monitoring tells us that a known condition is true; observability helps us ask new questions during an incident. Treat telemetry as a product: define users and decisions, instrument consistently, control cost, and test the path from code to notification.

### A simple mental model

Logs describe individual events, metrics summarize behavior over time, traces connect the steps of one request, and profiles show where a process spends CPU or memory. Start with metrics to detect a change, then use traces, logs, and profiles to explain it.

## The signals

| Signal | Best question | Typical data |
| --- | --- | --- |
| Logs | What happened, with context? | Structured events and errors |
| Metrics | How often and how much? | Counters, gauges, histograms |
| Traces | Where did time and failure go? | Spans linked by trace context |
| Profiles | Which code consumed resources? | CPU, allocation, lock and heap samples |
| Events | What changed? | Deployments, feature flags, scaling |

Use all signals together. A checkout latency SLO alert starts in metrics, a trace identifies the slow payment call, and a structured log supplies the provider response. A profile explains CPU regressions that do not appear in a single request.

## A practical architecture

Application SDKs emit OpenTelemetry data to a local agent or gateway. Collectors batch, enrich, redact, sample, and route to independent backends for metrics, logs, traces, and profiles. Dashboards answer recurring questions; alerts page only when a person can take action. Keep a small, documented set of stable resource attributes (`service.name`, `service.version`, `deployment.environment`, region) across every signal.

## Instrumentation checklist

1. Name the user journey and its SLI before adding spans or dashboards.
2. Instrument framework boundaries, outbound calls, queues, database queries, and background jobs.
3. Propagate W3C trace context across HTTP, messaging, and asynchronous work.
4. Record duration and outcome; attach bounded, low-cardinality dimensions.
5. Never put secrets, tokens, payment data, or raw user content in telemetry.
6. Make telemetry failure non-fatal, bounded in memory, and observable itself.
7. Test representative requests and verify data reaches the backend in staging.

## Trade-offs

More telemetry improves diagnosis but costs storage, network, CPU, and attention. Verbose logs hide important events, high-cardinality labels make metrics expensive, and aggressive trace sampling can miss rare failures. Begin with the smallest data set that answers a real operational question; add detail when an incident or SLO review proves it is needed.

## Guides

- [Logging](logging.md) · [Metrics](metrics.md) · [Tracing](tracing.md) · [Distributed tracing](distributed-tracing.md)
- [Alerting](alerting.md) · [SLIs, SLOs, and SLAs](slo-sli-sla.md)
- [OpenTelemetry and telemetry pipelines](opentelemetry.md) · [Dashboards and runbooks](dashboards.md)
- [Profiling](profiling.md) · [Frontend and synthetic monitoring](frontend-and-synthetic.md)
- [Telemetry security and cost](telemetry-security-cost.md) · [Incident response](incident-observability.md)

## Useful references

- [OpenTelemetry concepts](https://opentelemetry.io/docs/concepts/) and [Collector configuration](https://opentelemetry.io/docs/collector/configuration/)
- [Prometheus naming](https://prometheus.io/docs/practices/naming/) and [histograms](https://prometheus.io/docs/practices/histograms/)
- [W3C Trace Context](https://www.w3.org/TR/trace-context/)
- [Google SRE: alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
