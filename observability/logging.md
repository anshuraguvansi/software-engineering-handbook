# Logging

Logs are an event stream, not a debugging printout. Emit one structured record per meaningful event and let a collector add service and environment metadata.Logging records discrete events produced by an application or system. A log entry should answer what happened, when it happened, where it happened, and whether it succeeded.

## Why we need it

Logs preserve the details that aggregated metrics cannot: an exception type, a provider response, or the state transition that preceded a failure. They are especially useful for investigating one request or a rare event.

## How it should be implemented

Use structured fields, consistent severity levels, UTC timestamps, and a request or trace ID. Define a schema and redact sensitive data before export.

## Event shape

```json
{
  "timestamp": "2026-09-05T14:03:11.421Z",
  "level": "WARN",
  "message": "payment provider timed out",
  "service.name": "checkout",
  "service.version": "2026.09.05.2",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7",
  "route": "POST /v1/orders",
  "provider": "acme-payments",
  "duration_ms": 2000,
  "error.type": "TimeoutError"
}
```

For example, this entry records a payment timeout while allowing an engineer to find the complete request through `trace_id`.

Use a stable schema, UTC timestamps, severity levels with documented meanings, and correlation IDs. Put the human-readable explanation in `message`; put queryable facts in fields. Log at boundaries and state transitions, including job IDs and attempt numbers for asynchronous work.

## Levels and volume

`DEBUG` is local or sampled diagnostic detail. `INFO` records normal business milestones. `WARN` means degraded behavior or a recoverable anomaly. `ERROR` means an operation failed and needs investigation. `FATAL` should be rare and process-ending. Avoid logging every successful request at high volume if a metric already answers the question.

## Privacy and reliability

Allow-list fields and redact at the logger boundary. Hash identifiers only when support workflows need correlation; hashing is not encryption. Never log credentials, authorization headers, full card numbers, health data, or unrestricted request bodies. Use bounded queues, backpressure, local buffering, and a drop counter so an exporter outage cannot take down the service.

## Query patterns

- Find one request: `trace_id`, then inspect its spans and logs.
- Find an incident: filter by service, version, region, and error class; compare with a baseline.
- Find a customer impact: use a support-safe request or account correlation ID, never a sensitive value.

Prefer metrics for alert thresholds. A log-derived metric is useful for rare events, but document the parsing rule and test it against schema changes.

## Trade-offs and when to use logs

Plain text is easy to read locally; JSON is easier to search in production. Logging every detail helps an investigation but increases cost and privacy risk. Log once at the layer that owns the decision. Use logs for individual examples and unusual state transitions; use metrics for rates and traces for one request's complete path.
