# Frontend and synthetic monitoring

## What is frontend monitoring?

Frontend monitoring measures the experience of people using a website or mobile application. It covers the code running in the browser or on the device, the network between that client and the backend, and the result shown to the user.

There are two main forms:

- **Real-user monitoring (RUM):** collects measurements from real browser and mobile sessions.
- **Synthetic monitoring:** runs scripted tests from controlled browsers, devices, and locations.

## Why do we need it?

Backend dashboards can show that an API is healthy while users still experience a broken page. A CDN, DNS provider, JavaScript bundle, third-party script, device, or regional network can fail before the request reaches the backend.

Frontend monitoring helps answer questions such as:

- Can users load the application?
- How long does the page take to become usable?
- Which browsers, devices, versions, or regions are affected?
- Are JavaScript errors preventing a button or screen from working?
- Did a release make the application slower?

## Browser monitoring

Browser RUM commonly collects:

- page-load and navigation timing;
- Core Web Vitals such as LCP, INP, and CLS;
- JavaScript exceptions and unhandled promise rejections;
- failed resources such as scripts, stylesheets, fonts, and images;
- API request duration and status;
- browser, operating system, device class, route, and region.

Group results by bounded values such as browser family and major version. Do not collect passwords, authorization tokens, full URLs containing secrets, or unrestricted page content. Obtain consent where applicable and provide a way to remove user identifiers.

Example browser event:

```json
{
  "event": "api_error",
  "route": "/checkout",
  "browser": "Chrome",
  "duration_ms": 2100,
  "status": 503,
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736"
}
```

## Mobile monitoring

Mobile applications need additional signals because they run on unreliable networks and many hardware and operating-system versions. Collect app start time, screen load time, API latency, crash and fatal-error reports, network failures, battery or resource pressure where useful, and app version.

Segment by operating-system version, device model, app version, network type, and region, while keeping the number of distinct values bounded. Upload crash reports after startup when possible so a network outage does not prevent diagnosis. Never record personal data or authentication tokens in crash metadata.

Mobile tools often include native crash reporting and performance monitoring. OpenTelemetry mobile support can add traces and metrics, but verify SDK maturity and battery, privacy, and data-transfer costs before enabling broad instrumentation.

## Connecting frontend and backend traces

The frontend starts a trace for a user action or receives a trace context from the page navigation. When it calls an instrumented backend API, the browser or mobile SDK injects W3C trace context into the request, usually through the `traceparent` header. The backend extracts it and creates a child server span. Backend services then propagate the same context to their dependencies.

```text
Browser span: click “Pay”
    ↓ traceparent (trace_id = T1)
API gateway span
    ↓ traceparent (trace_id = T1)
Checkout service span
    ↓
Payment service and database spans
```

The frontend event and backend spans can therefore be opened as one trace. Configure CORS to allow the tracing headers when the browser calls another origin, and ensure proxies preserve them. Do not expose internal secrets in baggage or tracing attributes. For mobile, the HTTP client instrumentation performs the same injection; background work should explicitly carry context when it outlives the screen or request.

## Synthetic monitoring

Synthetic tests repeatedly execute known journeys from controlled locations. A simple test might resolve DNS, establish TLS, load the home page, sign in with a test account, and call a health-safe API. More complete tests can exercise search or checkout with isolated, idempotent test data.

Record test location, browser or device profile, script version, step name, duration, and failure reason. Keep test credentials separate from real users and do not send real emails, payments, or notifications. Synthetic checks are useful during quiet periods, but they cannot represent every real device, network, or user cohort.

## Alerts and trade-offs

Alert on sustained user-impacting changes, such as a large increase in JavaScript errors or a failed critical journey. Compare synthetic results with RUM before paging so a broken test does not create an outage alert. Sampling reduces browser and mobile overhead but can hide rare failures; crash events and severe errors should usually be retained at a higher rate. Frontend telemetry also increases privacy, bandwidth, storage, and battery costs, so collect only data that supports a documented decision.

## Common tools

| Tool | Common use |
| --- | --- |
| Sentry | Browser and mobile errors, stack traces, releases, and performance traces |
| Datadog RUM | Browser and mobile sessions, Core Web Vitals, errors, and backend correlation |
| New Relic Browser and Mobile | Frontend performance, errors, sessions, and service relationships |
| Firebase Crashlytics | Crash and non-fatal error reporting for Android and iOS |
| Playwright or Cypress | Browser-based synthetic and end-to-end tests |
| WebPageTest or Lighthouse | Page performance analysis and lab measurements |

Choose tools based on supported platforms, privacy controls, backend integration, retention, cost, and whether you need hosted analysis or an OpenTelemetry-based pipeline.
