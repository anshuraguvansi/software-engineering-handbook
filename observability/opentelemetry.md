# OpenTelemetry

OpenTelemetry (OTel) is a collection of APIs, SDKs, instrumentation libraries, semantic conventions, and a Collector for generating and transporting logs, metrics, and traces. It is vendor-neutral, so application code does not need to depend directly on Datadog, New Relic, Jaeger, or another backend.

## Why do we need it?

Without a common telemetry standard, every service may use a different library and field name. OpenTelemetry gives services a consistent way to create spans, record metrics, correlate logs, propagate context, and change backends later.

## How does it work?

```text
Application
  └─ OpenTelemetry API and SDK
       └─ instrumentation and exporters
            └─ OpenTelemetry Collector
                 └─ metrics, logs, and trace backends
```

The **API** is what application code calls. The **SDK** records and exports the data. An **instrumentation library** adds telemetry to a framework or client automatically. The **Collector** receives, processes, and routes telemetry to one or more backends.

## Typical architecture

Telemetry can come from the application and from the infrastructure running it:

```text
Application code and libraries ──┐
                                  │
Hosts, containers, and Kubernetes ├──> OpenTelemetry Collector ───> Telemetry backend
                                  │       (receive, process, route)  (metrics, logs, traces)
AWS services and resources ──────┘
```

Application instrumentation provides request metrics, business metrics, logs, and traces. Infrastructure agents and cloud integrations provide CPU, memory, network, container, load balancer, database, and platform telemetry. The Collector can batch, redact, sample, enrich, and route both types of data to one or more backends.

## Automatic instrumentation example

Automatic instrumentation is the quickest way to cover supported HTTP servers, HTTP clients, databases, and messaging libraries. For Python, a typical setup is:

```bash
pip install opentelemetry-distro opentelemetry-exporter-otlp
opentelemetry-bootstrap -a install

OTEL_SERVICE_NAME=checkout-service \
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317 \
opentelemetry-instrument python app.py
```

The instrumentation starts server and client spans, propagates trace context, and exports them to the configured OTLP endpoint. The exact packages and startup command depend on the language and framework.

## Manual instrumentation example

Use manual instrumentation for business operations that automatic instrumentation cannot understand:

```python
from opentelemetry import trace

tracer = trace.get_tracer("checkout-service")

def create_order(order):
    with tracer.start_as_current_span("create-order") as span:
        span.set_attribute("order.item_count", len(order.items))
        validate_order(order)
        return save_order(order)
```

Keep span names and attributes bounded. Do not add passwords, tokens, full request bodies, or unlimited user content.

## Collector example

The Collector receives OTLP data, limits memory, batches records, adds a resource attribute, and exports the signals to a backend:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 512
  batch: {}
  resource:
    attributes:
      - key: deployment.environment
        value: production
        action: upsert

exporters:
  otlp/backend:
    endpoint: telemetry.example:443
    tls:
      insecure: false

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch, resource]
      exporters: [otlp/backend]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch, resource]
      exporters: [otlp/backend]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch, resource]
      exporters: [otlp/backend]
```

A **local agent** is usually deployed beside each application for short network paths and buffering. A **gateway** is a shared Collector used for central routing, redaction, sampling, and access control. Small systems may send directly to a backend, but a Collector makes policy and backend changes easier to manage.

## Production checklist

- Set `service.name`, `service.version`, and `deployment.environment` consistently.
- Use TLS and authentication between applications, Collectors, and backends.
- Configure batching, bounded queues, retries, and memory limits.
- Sample traces carefully; retain errors and important slow requests.
- Redact sensitive fields before export.
- Monitor received, exported, refused, and dropped records for every signal.
- Validate Collector configuration in CI and test telemetry in staging.
- Make telemetry failure non-fatal so an exporter outage cannot stop the application.
