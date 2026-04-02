---
name: monitoring
description: Logging, metrics (Prometheus), Grafana dashboards, distributed tracing, and alerting standards. Invoke when setting up observability, designing dashboards, writing log statements, or configuring alerts.
---

# Monitoring & Observability Skill

---

## 🔭 The Three Pillars of Observability

| Pillar | Tool | Purpose |
|--------|------|---------|
| **Logs** | Pino + Loki | What happened |
| **Metrics** | Prometheus + Grafana | How the system is behaving |
| **Traces** | OpenTelemetry + Jaeger | Why something is slow |

---

## 📝 Logging Rules

### Log Levels
| Level | When to Use |
|-------|-------------|
| `error` | Unexpected failure requiring attention |
| `warn` | Unexpected but recoverable situation |
| `info` | Normal significant events (startup, request lifecycle) |
| `debug` | Detailed debugging info (dev only) |
| `trace` | Very verbose (never in production) |

### Log Format — Structured JSON (ALWAYS)
```js
// ✅ Structured log — searchable and parseable
logger.info({
  event: 'order.placed',
  orderId: order.id,
  userId: user.id,
  amount: order.total,
  durationMs: Date.now() - startTime,
  requestId: req.id,
});

// ❌ Unstructured log — cannot be queried
console.log(`Order ${orderId} placed by user ${userId}`);
```

### Mandatory Fields
```js
{
  level: 'info',
  timestamp: '2026-01-01T00:00:00.000Z',
  service: 'order-service',
  version: '1.2.3',
  environment: 'production',
  requestId: 'uuid',       // trace across services
  userId: 'uuid',          // who triggered it
  event: 'order.placed',   // what happened
  durationMs: 45,          // how long
}
```

### What NOT to Log
```js
// ❌ NEVER log sensitive data
logger.info({ password: user.password });           // FORBIDDEN
logger.info({ token: req.headers.authorization });  // FORBIDDEN
logger.info({ creditCard: payment.card });          // FORBIDDEN
```

### Logger Setup (Pino)
```js
// src/utils/logger.js
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  base: {
    service: process.env.APP_NAME,
    version: process.env.npm_package_version,
    env: process.env.NODE_ENV,
  },
  timestamp: pino.stdTimeFunctions.isoTime,
});
```

---

## 📊 Metrics (Prometheus)

### Metric Types
| Type | Use Case | Example |
|------|----------|---------|
| **Counter** | Values that only increase | `http_requests_total` |
| **Gauge** | Values that go up and down | `active_connections` |
| **Histogram** | Distribution of values | `http_request_duration_seconds` |
| **Summary** | Pre-calculated percentiles | `request_latency_percentiles` |

### Naming Convention
```
# Pattern: {namespace}_{subsystem}_{name}_{unit}
http_request_duration_seconds    # histogram
http_requests_total              # counter
http_requests_in_flight          # gauge
db_query_duration_seconds        # histogram
cache_hits_total                 # counter
cache_misses_total               # counter
queue_messages_pending           # gauge
```

### Labels — Low Cardinality Only
```js
// ✅ Use labels for meaningful dimensions
httpRequestCounter.labels({
  method: req.method,
  route: '/api/v1/users',    // normalized route, not userId!
  status_code: res.statusCode,
  service: 'user-service',
}).inc();

// ❌ Don't use high-cardinality labels (userId, orderId)
// This creates millions of time series → kills Prometheus
```

### Express Metrics Middleware
```js
import client from 'prom-client';

const httpDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
});

export function metricsMiddleware(req, res, next) {
  const end = httpDuration.startTimer();
  res.on('finish', () => {
    end({ method: req.method, route: req.route?.path || req.path, status_code: res.statusCode });
  });
  next();
}

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});
```

---

## 📈 Grafana Dashboard Design

### The RED Method (for every service dashboard)
Every service dashboard MUST have:
- **R** — **Rate**: requests per second
- **E** — **Errors**: error rate (%)
- **D** — **Duration**: P50, P95, P99 latency

```promql
# Rate
rate(http_requests_total{service="order-service"}[5m])

# Error rate
rate(http_requests_total{status_code=~"5.."}[5m])
/ rate(http_requests_total[5m]) * 100

# P99 latency (ms)
histogram_quantile(0.99,
  rate(http_request_duration_seconds_bucket{service="order-service"}[5m])
) * 1000
```

### Standard Dashboard Layout
```
Row 1: Summary / Health Overview (traffic lights)
Row 2: RED metrics (Rate, Errors, Duration)
Row 3: Resource usage (CPU, Memory, DB connections)
Row 4: Business metrics (orders/min, signups, payments)
Row 5: Logs panel (Loki integration)
```

---

## 🚨 Alerting Rules

### Severity Levels
| Level | Response Time | Example |
|-------|--------------|---------|
| `critical` | Immediate (PagerDuty) | Service down, payment failures |
| `warning` | Within 30min (Slack) | High error rate, slow queries |
| `info` | Business hours | Unusual traffic patterns |

### Standard Alert Rules (AlertManager)
```yaml
groups:
  - name: service-alerts
    rules:
      - alert: ServiceDown
        expr: up{job="my-service"} == 0
        for: 1m
        severity: critical
        annotations:
          summary: "Service {{ $labels.job }} is down"

      - alert: HighErrorRate
        expr: |
          rate(http_requests_total{status_code=~"5.."}[5m])
          / rate(http_requests_total[5m]) > 0.05
        for: 5m
        severity: warning
        annotations:
          summary: "Error rate > 5% on {{ $labels.service }}"

      - alert: HighLatency
        expr: |
          histogram_quantile(0.99,
            rate(http_request_duration_seconds_bucket[5m])
          ) > 1
        for: 5m
        severity: warning
        annotations:
          summary: "P99 latency > 1s on {{ $labels.service }}"
```

### Alert Naming Convention
```
{Severity}{Service}{Problem}
CriticalPaymentServiceDown
WarningOrderServiceHighLatency
WarningRedisLowHitRate
CriticalDatabaseConnectionsExhausted
```

---

## 🔍 Distributed Tracing (OpenTelemetry)

```js
// src/tracing.js
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';

const sdk = new NodeSDK({
  traceExporter: new JaegerExporter({ endpoint: process.env.JAEGER_ENDPOINT }),
  instrumentations: [getNodeAutoInstrumentations()],
  serviceName: process.env.APP_NAME,
});

sdk.start();
```

### Span Naming Convention
```
# HTTP: {method} {route}
GET /api/v1/users/:id

# DB: {operation} {table}
SELECT users
INSERT orders

# Cache: {operation} {key_pattern}
GET user:{id}
SET session:{id}

# Queue: {operation} {queue_name}
PUBLISH order.placed
CONSUME email.send
```
