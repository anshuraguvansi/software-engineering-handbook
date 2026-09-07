# Alerting

Alerting checks telemetry for a problem and tells the right owner when attention is needed. Send an urgent page only when a person must act soon to protect users or meet a reliability goal. Send less urgent problems as tickets, and keep informational events in dashboards or logs without notifying anyone.

## What is a notification?

A notification is the message delivered after an alert rule fires. It tells a person or system that a condition needs attention. The alert is the detected condition; the notification is how that condition is communicated.

For example:

```text
Alert condition: checkout error budget is burning at 14.4×
Notification: PagerDuty page sent to the checkout on-call engineer
Response: investigate the linked dashboard and follow the runbook
```

Common notification destinations include:

- **Page:** an urgent push, phone call, or SMS for immediate action.
- **Ticket:** an issue in a work-tracking system for non-urgent follow-up.
- **Chat message:** a message in Slack or Teams for team awareness or coordination.
- **Email:** useful for summaries and low-priority information, but easy to miss during an outage.
- **Webhook:** a machine-readable request to another system, such as an automation or incident-management tool.

Route notifications using severity, service ownership, environment, and time of day. Group duplicate alerts, suppress dependent symptoms, and include the alert name, impact, start time, current value, owner, runbook link, and dashboard link. Never send the same event to every channel by default; unnecessary notifications create noise and cause important pages to be ignored.

## Why we need it

People cannot watch dashboards continuously. Good alerts reduce detection time for user-impacting failures without creating so much noise that on-call engineers ignore them.

## How it should be implemented

Start by alerting on a user-impacting symptom, such as a service using its SLO error budget too quickly. Do not page simply because a CPU or memory graph looks high; those signals are useful clues, but the alert should indicate that users may be affected.

Every alert should define:

- **Condition:** what must be true, such as the percentage of failed requests.
- **Window:** how long the condition is measured, such as the last five minutes.
- **Severity:** how quickly someone must respond.
- **Owner:** the team responsible for investigating it.
- **Runbook and dashboard:** links that explain what to check and how to respond.
- **Resolve condition:** the rule that confirms the problem has stopped.

### What is burn rate?

An SLO gives a service an allowed amount of failure called an **error budget**. For a 99.9% availability SLO, the allowed failure rate is 0.1%. **Burn rate** tells us how quickly the service is spending that allowance compared with the planned rate:

```text
burn rate = observed error rate / allowed error rate
```

For example, if the service normally may have 0.1% errors but currently has 1% errors, its burn rate is 10×. At that rate, it will use the monthly error budget roughly ten times faster than planned. A high burn rate should page someone; a low but sustained burn rate may create a ticket.

Use two windows for an urgent alert. The long window confirms that the problem is significant, while the short window confirms that it is still happening. The expression below pages only when both the one-hour and five-minute error rates show a 14.4× burn rate. This reduces pages caused by a short, already-resolved spike.

Alert on symptoms (SLO error budget burn, unavailable checkout) and use causes (CPU, queue depth) as supporting signals. Avoid paging on every host or pod when the service-level symptom is healthy.

## Burn-rate example

For a 99.9% 30-day availability SLO, start with multi-window, multi-burn-rate alerts: page at 14.4× over 1 hour and 5 minutes, or 6× over 6 hours and 30 minutes; create a ticket around 1× over 3 days and 6 hours. Tune for traffic, impact, and response time. Low-traffic services may need synthetic checks or longer windows.

```promql
(
  slo:error_ratio:rate1h > 14.4 * 0.001
  and slo:error_ratio:rate5m > 14.4 * 0.001
)
or (
  slo:error_ratio:rate6h > 6 * 0.001
  and slo:error_ratio:rate30m > 6 * 0.001
)
```

Review alert precision, recall, detection time, reset time, pages per on-call, and acknowledged-without-action counts every month. Silence is a controlled, time-bounded incident action; never use it to hide a broken rule.

## Trade-offs and mistakes

Lower thresholds detect problems sooner but create noise. Higher thresholds reduce pages but can miss a slow, sustained failure. Page on user impact and error-budget burn; keep CPU, memory, and queue alerts as clues unless they predict impact. Test notification delivery and recovery, not only the firing condition.
