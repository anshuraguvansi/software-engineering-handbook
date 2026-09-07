# Service Level Indicators, Objectives, and Agreements

These three terms describe different parts of reliability measurement:

- **SLI — Service Level Indicator:** the actual measurement of a service's behavior.
- **SLO — Service Level Objective:** the reliability target the team aims to achieve.
- **SLA — Service Level Agreement:** a formal promise to customers, often with reporting requirements or financial consequences.

The SLI is the result, the SLO is the engineering target, and the SLA is the external commitment. An SLA may use an SLO as its target, but the two do not have to be identical.

## Why we need them

They turn vague reliability goals into measurable decisions. The error budget shows how much unreliability is acceptable and helps teams balance feature delivery with reliability work.

## How they should be implemented

Define good and total events, eligibility, latency thresholds, exclusions, data source, and aggregation before measuring. Publish the target, current performance, budget remaining, and the policy for spending that budget.

## Define an SLI

Start with the user journey and eligibility rules. For checkout availability:

```text
good events = requests that return a valid order confirmation in ≤ 2 seconds
total events = eligible checkout requests, excluding explicitly documented client cancellations
SLI = good events / total events
```

Specify inclusion, latency threshold, status mapping, aggregation, and data source. Measure at the boundary users experience; server CPU is a diagnostic metric, not an availability SLI.

## Complete example

Suppose a team runs a payment API.

```text
SLI (measurement):
  998,900 successful requests / 1,000,000 eligible requests
  = 99.89% availability

SLO (internal target):
  At least 99.9% of eligible requests succeed each calendar month

SLA (customer promise):
  At least 99.5% monthly availability, with a service credit if the target is missed
```

The service met its SLA (99.89% is above 99.5%) but missed its internal SLO (99.89% is below 99.9%). The team should investigate and prioritize reliability work even though a customer credit is not required.

For a latency objective, the definitions might be:

```text
SLI: 99.2% of eligible requests complete within 500 ms
SLO: at least 99% complete within 500 ms each month
SLA: at least 95% complete within 1 second each month
```

Latency SLIs should define the threshold and percentile or good-event ratio. “Average latency” alone can hide slow requests at the tail.

## Error budgets and policy

For a 99.9% monthly SLO, the error budget is 0.1% of eligible events (about 43m 12s of a 30-day month for an availability interpretation). Budget burn is observed error rate divided by the allowed error rate. Define policy before an incident: when budget is low, pause risky launches, prioritize reliability work, or require an explicit product decision. A budget is a prioritization mechanism, not permission to spend failures deliberately.

Review SLOs with product and support teams. Segment by region, tier, and critical journey when one aggregate hides harm, but avoid so many objectives that teams cannot reason about them. Publish current performance, budget remaining, and exclusions.

## Trade-offs and mistakes

A strict SLO encourages reliability but can slow feature delivery and increase cost. A loose SLO leaves room to ship but may hide poor customer experience. Choose a small number of journeys that matter, define exclusions before measuring, and never change the definition only after an incident. An SLA may require a different measurement or reporting process than the engineering SLO.
