# Telemetry security and cost

Telemetry is production data. Classify fields, minimize collection, encrypt in transit and at rest, apply least-privilege access, and define retention and deletion policies. Separate tenants and environments. Audit queries and exports. Redact before data leaves the process because backend filters are too late.

## What this covers

Telemetry security protects the data emitted by applications. Telemetry cost management controls the volume, retention, and processing expense of that data.

## Why we need it

Logs and traces can contain sensitive information and can grow faster than application data. A leak or an uncontrolled label can create both compliance and financial incidents.

## How it should be implemented

Classify fields, allow-list attributes, redact at the source, restrict access, set retention tiers, bound labels, and measure volume by service and signal.

Control cost with bounded labels, aggregation, log-level policies, dynamic sampling, tail sampling for errors and slow traces, retention tiers, and quotas. Track bytes and events by service, signal, environment, and exporter. Alert on sudden volume changes and dropped-data ratios. Do not reduce SLO metrics or incident evidence merely to hit a budget; reduce redundant detail first.
