---
name: bun-server-health-metrics
description: Health checks and metrics guide for Bun Server framework. Use when implementing health endpoints, liveness/readiness probes, Prometheus metrics, application monitoring, or observability.
---

# Bun Server Health & Metrics

## Health Module

### Setup HealthModule

```typescript
import { Module, HealthModule, DatabaseHealthIndicator } from "@dangao/bun-server";

HealthModule.forRoot({
  path: "/health",
  indicators: [DatabaseHealthIndicator],
});

@Module({
  imports: [HealthModule, DatabaseModule],
})
class AppModule {}
```

### Health Endpoint Response

```json
// GET /health
{
  "status": "up",
  "details": {
    "database": {
      "status": "up",
      "responseTime": 5
    }
  },
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```

### Custom Health Indicator

```typescript
import { Injectable, HealthIndicator, HealthIndicatorResult } from "@dangao/bun-server";

@Injectable()
class RedisHealthIndicator implements HealthIndicator {
  constructor(private readonly redis: RedisService) {}

  async check(): Promise<HealthIndicatorResult> {
    try {
      const start = Date.now();
      await this.redis.ping();
      const responseTime = Date.now() - start;

      return {
        name: "redis",
        status: "up",
        details: { responseTime },
      };
    } catch (error) {
      return {
        name: "redis",
        status: "down",
        details: { error: error.message },
      };
    }
  }
}

// Register
HealthModule.forRoot({
  indicators: [DatabaseHealthIndicator, RedisHealthIndicator],
});
```

### External Service Health Check

```typescript
@Injectable()
class PaymentServiceHealthIndicator implements HealthIndicator {
  async check(): Promise<HealthIndicatorResult> {
    try {
      const response = await fetch("https://payment.example.com/health");
      return {
        name: "payment-service",
        status: response.ok ? "up" : "down",
      };
    } catch (error) {
      return {
        name: "payment-service",
        status: "down",
        details: { error: error.message },
      };
    }
  }
}
```

## Kubernetes Probes

```typescript
HealthModule.forRoot({
  path: "/health",
  livenessPath: "/health/live",
  readinessPath: "/health/ready",
  indicators: [DatabaseHealthIndicator],
});

// Liveness: Is the app running?
// GET /health/live -> { "status": "up" }

// Readiness: Can it accept traffic?
// GET /health/ready -> { "status": "up", "details": {...} }
```

## Metrics Module

### Setup MetricsModule

```typescript
import { Module, MetricsModule } from "@dangao/bun-server";

MetricsModule.forRoot({
  path: "/metrics",
  defaultLabels: {
    app: "my-service",
    env: process.env.NODE_ENV,
  },
});

@Module({
  imports: [MetricsModule],
})
class AppModule {}
```

### Prometheus Metrics Output

```
# GET /metrics
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",path="/api/users",status="200"} 150
http_requests_total{method="POST",path="/api/users",status="201"} 23

# HELP http_request_duration_seconds HTTP request duration
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.1"} 120
http_request_duration_seconds_bucket{le="0.5"} 145
http_request_duration_seconds_bucket{le="1"} 150
```

### HTTP Metrics Middleware

```typescript
import { createHttpMetricsMiddleware } from "@dangao/bun-server";

const app = new Application({ port: 3100 });
app.use(createHttpMetricsMiddleware());
```

### Custom Metrics

```typescript
import {
  Injectable,
  Inject,
  MetricsCollector,
  METRICS_SERVICE_TOKEN,
} from "@dangao/bun-server";

@Injectable()
class OrderService {
  constructor(
    @Inject(METRICS_SERVICE_TOKEN) private readonly metrics: MetricsCollector
  ) {}

  async createOrder(data: CreateOrderDto) {
    const timer = this.metrics.startTimer("order_processing_duration");

    try {
      const order = await this.processOrder(data);

      // Counter
      this.metrics.increment("orders_created_total", {
        type: data.type,
        status: "success",
      });

      // Gauge
      this.metrics.set("orders_pending", await this.getPendingCount());

      timer.end({ status: "success" });
      return order;
    } catch (error) {
      this.metrics.increment("orders_created_total", {
        type: data.type,
        status: "failed",
      });
      timer.end({ status: "failed" });
      throw error;
    }
  }
}
```

### Metric Types

```typescript
// Counter - monotonically increasing
this.metrics.increment("requests_total");
this.metrics.increment("errors_total", { type: "validation" });

// Gauge - can go up and down
this.metrics.set("active_connections", 42);
this.metrics.increment("active_connections");
this.metrics.decrement("active_connections");

// Histogram - distribution of values
this.metrics.observe("request_duration_seconds", 0.25);
this.metrics.observe("response_size_bytes", 1024, { endpoint: "/api/users" });

// Timer helper
const timer = this.metrics.startTimer("operation_duration");
// ... do work ...
timer.end({ status: "success" });
```

### Prometheus Formatter

```typescript
import { PrometheusFormatter, MetricsCollector } from "@dangao/bun-server";

const metrics = new MetricsCollector();
const formatter = new PrometheusFormatter();

// Get Prometheus format
const output = formatter.format(metrics.getMetrics());
```

## Monitoring Patterns

### Request Tracing

```typescript
function requestTracingMiddleware(ctx: Context, next: NextFunction) {
  const requestId = ctx.req.headers.get("x-request-id") || crypto.randomUUID();
  const start = performance.now();

  ctx.set("requestId", requestId);

  const response = next();
  const duration = performance.now() - start;

  // Log with request ID
  console.log(JSON.stringify({
    requestId,
    method: ctx.req.method,
    path: new URL(ctx.req.url).pathname,
    duration,
    status: response instanceof Response ? response.status : 200,
  }));

  return response;
}
```

### Error Rate Monitoring

```typescript
@Injectable()
class ErrorMetricsService {
  constructor(
    @Inject(METRICS_SERVICE_TOKEN) private readonly metrics: MetricsCollector
  ) {}

  recordError(error: Error, context: { endpoint: string; method: string }) {
    this.metrics.increment("errors_total", {
      type: error.name,
      endpoint: context.endpoint,
      method: context.method,
    });
  }
}
```

## Related Resources

- [Performance Guide](../../bun-server/docs/performance.md)
- [Deployment Guide](../../bun-server/docs/deployment.md)
