# Continuous profiling

Profiling is a type of telemetry. It is different from a log, metric, or trace because it samples what the running process is doing inside its code rather than recording an event or a single request. Profiling periodically samples stack traces and records how much CPU, memory, or waiting time code consumes.

## Why we need it

Aggregated metrics show that CPU or memory is high, while a profile identifies the functions responsible.

## How it should be used

Collect low-overhead profiles continuously, compare releases under the same workload, and increase sampling only during a controlled investigation. Profiling tools periodically sample stack traces; they do not pause the application and inspect every instruction.

## When does profiling happen?

Profiling can happen in three ways:

- **Continuously:** a low sampling rate runs in production all the time, making it possible to compare normal behavior with a later incident.
- **On demand:** an engineer increases sampling or captures a profile after a CPU, memory, latency, or lock-contention alert.
- **During testing:** a profile is captured during a benchmark, load test, or canary deployment to compare versions before a wider release.

For example:

```text
Metric: CPU increased from 45% to 90%
Profile: JSON parsing uses 65% of CPU samples
Action: optimize parsing and compare the new release with the baseline
```

Never collect raw secrets or user payloads in profile labels. Restrict profile access because stack traces and symbol names can reveal internals.

Correlate a profile window with deploys, SLO burn, host saturation, and trace exemplars. A profile explains aggregate cost; a trace explains one request. Use both before changing code.
