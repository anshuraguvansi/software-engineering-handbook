# Metrics

Metrics are named measurements recorded over time. They help us understand how often something happens, how much work is in progress, or how a value is distributed. Counters increase, gauges represent a current value, and histograms record a distribution such as request latency.

## Why we need them

Metrics make system behavior visible without storing every individual event. The most important uses are:

- **Monitor health:** track CPU, memory, disk space, database connections, and queue depth.
- **Detect user-impacting problems:** watch request errors, availability, and latency.
- **Create alerts:** notify the team when an error rate, latency, or resource value crosses a safe limit.
- **Measure SLOs:** calculate availability, latency objectives, and remaining error budget.
- **Investigate incidents:** identify when a problem started, how large it is, and which service or region is affected.
- **Plan capacity:** understand traffic growth and decide when to add compute, storage, database, or queue capacity.
- **Compare releases:** compare error rate, latency, and resource usage before and after a deployment.
- **Track business activity:** measure events such as orders created, payments completed, sign-ups, and messages delivered.
- **Drive automation:** use request rate, queue depth, or resource usage to autoscale workers and services.

For example, a sudden increase in `http_request_duration_seconds` may trigger an alert, show that an SLO is at risk, and help an engineer compare the current deployment with the previous one. Metrics usually detect the problem; logs and traces provide the details about an individual request.

## How they should be implemented

Choose a type that matches the question, name it with a unit, and use only bounded labels. For example, record `http_requests_total` with `route`, `method`, and `status_class`, rather than a user ID or raw URL.

Example instrumentation for an HTTP service might produce:

```text
http_requests_total{service="checkout",route="/orders",method="POST",status_class="2xx"} 12500
http_request_duration_seconds_count{service="checkout",route="/orders"} 12500
http_request_duration_seconds_sum{service="checkout",route="/orders"} 3125.4
checkout_orders_created_total{service="checkout",region="us-east"} 8400
checkout_queue_depth{service="checkout",queue="payment"} 42
```

The counter says how many requests completed, the histogram tracks their duration, the business counter records successful orders, and the gauge shows the current queue depth.

## How metrics are collected

The metric library records values inside the application. A separate collector or backend then gathers those values. There are two common collection models.

### Pull model

The application exposes a metrics endpoint, commonly `/metrics`. A system such as Prometheus periodically requests that endpoint:

```text
Prometheus → GET /metrics → Application
```

The application does not need to know when Prometheus will ask for data. The metric library keeps the current counter, gauge, and histogram values ready to be read.

### Push model

The application or its OpenTelemetry SDK exports metrics to a Collector or backend:

```text
Application → OpenTelemetry Collector → Metrics backend
```

The exporter sends values periodically or in batches. This model is useful when the application cannot expose an endpoint, such as in some serverless or short-lived jobs.

### Who is responsible for what?

```text
Application code       records measurements
Metric library / SDK   keeps and exports measurements
Collector or Prometheus gathers and processes them
Metrics backend        stores and queries time series
```

Choose one collection path for a metric. Collecting the same metric through both a Prometheus scrape and a push exporter can create duplicate series and incorrect dashboards. Monitor exporter queues, scrape failures, and dropped samples so problems in the telemetry path are visible.

## Types

- **Counter:** a total that only increases until the process restarts, such as requests, errors, or messages processed. Derive a per-second rate with `rate()`.
- **Gauge:** a value that can increase or decrease, such as queue depth, active connections, or memory currently used.
- **Histogram:** observations placed into buckets, such as request latency or response size. Compute percentiles and threshold percentages in the backend.
- **Summary:** client-side quantiles. It can be useful for a single process, but values are difficult to aggregate across replicas.

Do not use a counter for the current queue size or a gauge for a lifetime total. Do not calculate an average latency from only a p95 value; retain count and sum or use a histogram.

Name metrics with the unit and type where practical: `request_duration_seconds`, `queue_messages`, `bytes_total`. Labels should describe a small, stable set such as method, route template, status class, and region. Never label by user ID, request ID, raw URL, exception message, or unbounded tenant name.

## RED and USE

For request services use **Rate, Errors, Duration**. For resources use **Utilization, Saturation, Errors**. Add business metrics (orders accepted, payments declined) to connect technical health to user value.

```promql
# Error ratio for an SLO
sum(rate(http_requests_total{service="checkout",status_class="5xx"}[5m]))
/
sum(rate(http_requests_total{service="checkout"}[5m]))

# p95 latency from a classic histogram
histogram_quantile(0.95,
  sum(rate(http_request_duration_seconds_bucket{service="checkout"}[5m])) by (le, route))

# Requests per second by route
sum(rate(http_requests_total{service="checkout"}[5m])) by (route)

# Current payment queue depth
checkout_queue_depth{service="checkout",queue="payment"}
```

Choose histogram buckets around the SLO threshold and expected tail. Native histograms can reduce bucket design work where supported. Record rules for expensive expressions, and test them with representative traffic. Monitor scrape failures, missing series, stale timestamps, and exporter queue depth as telemetry health metrics.

## Trade-offs and when to use metrics

Metrics aggregate well and are excellent for trends, capacity, and alerts, but they lose individual-event detail. A `route` label is usually safe; a `user_id` label can create millions of time series. Use counters for totals, gauges for current values, and histograms for distributions. Pair metrics with exemplars or traces when someone needs to inspect a specific request.
