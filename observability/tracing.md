# Tracing

A trace is a tree (or directed graph) of spans representing one operation. A span has a start and end time, operation name, attributes, events, status, and links to related work. A trace follows one operation through the services, databases, queues, and external APIs it touches. Each timed segment is a span.

## Why we need it

Metrics can show that latency increased, but not which dependency caused it. A trace exposes the critical path, retries, errors, and time spent in each component.

## How it should be implemented

Create spans at inbound and outbound boundaries and for important domain operations. Use stable names and semantic attributes; attach IDs as attributes only when they are safe and bounded.

## Span design

Create spans at inbound/outbound boundaries and meaningful domain operations. Names describe the operation with bounded values (`HTTP POST`, `SELECT orders`, `charge`) rather than IDs. Attributes follow OpenTelemetry semantic conventions. Set `ERROR` status and record an exception when the operation failed; do not mark expected business outcomes such as a declined card as infrastructure errors unless the SLO treats them as failures.

```python
with tracer.start_as_current_span("charge") as span:
    span.set_attribute("payment.provider", provider)
    try:
        result = gateway.charge(order.total)
    except TimeoutError as exc:
        span.record_exception(exc)
        span.set_status(Status(StatusCode.ERROR, "provider timeout"))
        raise
```

## Sampling and cost

Head sampling makes an early keep/drop decision and keeps traces coherent when the decision propagates. Tail sampling in a Collector can retain all errors, slow traces, or selected tenants after seeing the whole trace. Keep metrics unsampled for SLOs. Sample sensitive attributes separately from the trace decision.

## Trace quality

Check clock synchronization, parent/child relationships, span duration, missing instrumentation, and exporter queue health. Add span links for batch consumers and fan-in/fan-out work where one parent is not accurate. Use exemplars to link a metric observation to a trace without putting trace IDs in metric labels.

## Trade-offs and when to use traces

Tracing gives per-request detail, but recording every span costs CPU, network, and storage. Sampling lowers cost and may hide rare failures. Use stable span names, avoid IDs in names, and instrument library boundaries before adding many tiny internal spans. Use traces to explain latency, retries, errors, and dependency behavior; use metrics for long-term counts and SLO calculations.
