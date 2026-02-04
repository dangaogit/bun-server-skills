---
name: bun-server-microservice
description: Microservice guide for Bun Server framework. Use when building microservices, service discovery, config center, load balancing, circuit breaker, distributed tracing, or inter-service communication.
---

# Bun Server Microservice

## Config Center (Nacos)

### Setup ConfigCenterModule

```typescript
import { Module, ConfigCenterModule } from "@dangao/bun-server";

ConfigCenterModule.forRoot({
  provider: "nacos",
  nacos: {
    serverAddr: "localhost:8848",
    namespace: "dev",
    group: "DEFAULT_GROUP",
    dataId: "my-service",
  },
});

@Module({
  imports: [ConfigCenterModule],
})
class AppModule {}
```

### Use Config Center

```typescript
import {
  Injectable,
  Inject,
  CONFIG_CENTER_TOKEN,
  ConfigCenter,
} from "@dangao/bun-server";

@Injectable()
class AppConfig {
  constructor(
    @Inject(CONFIG_CENTER_TOKEN) private readonly configCenter: ConfigCenter
  ) {}

  async getConfig(key: string) {
    return this.configCenter.get(key);
  }

  async watchConfig(key: string, callback: (value: any) => void) {
    return this.configCenter.subscribe(key, callback);
  }
}
```

## Service Registry (Nacos)

### Setup ServiceRegistryModule

```typescript
import { Module, ServiceRegistryModule } from "@dangao/bun-server";

ServiceRegistryModule.forRoot({
  provider: "nacos",
  nacos: {
    serverAddr: "localhost:8848",
    namespace: "dev",
  },
  service: {
    name: "user-service",
    port: 3100,
    ip: "192.168.1.100",
    metadata: {
      version: "1.0.0",
    },
  },
});

@Module({
  imports: [ServiceRegistryModule],
})
class AppModule {}
```

### Discover Services

```typescript
import {
  Injectable,
  Inject,
  SERVICE_REGISTRY_TOKEN,
  ServiceRegistry,
} from "@dangao/bun-server";

@Injectable()
class ServiceDiscovery {
  constructor(
    @Inject(SERVICE_REGISTRY_TOKEN) private readonly registry: ServiceRegistry
  ) {}

  async getOrderServiceInstances() {
    return this.registry.getInstances("order-service");
  }

  watchInstances(serviceName: string, callback: (instances: any[]) => void) {
    this.registry.subscribe(serviceName, callback);
  }
}
```

## Service Client

### Create Service Client

```typescript
import {
  ServiceClient,
  LoadBalancerFactory,
} from "@dangao/bun-server";

const orderClient = new ServiceClient({
  serviceName: "order-service",
  registry: serviceRegistry,
  loadBalancer: LoadBalancerFactory.create("round-robin"),
  timeout: 5000,
  retries: 3,
});

// Make calls
const order = await orderClient.get("/api/orders/123");
const created = await orderClient.post("/api/orders", { items: [...] });
```

### Load Balancing Strategies

```typescript
import {
  RandomLoadBalancer,
  RoundRobinLoadBalancer,
  WeightedRoundRobinLoadBalancer,
  ConsistentHashLoadBalancer,
  LeastActiveLoadBalancer,
} from "@dangao/bun-server";

// Random
const lb1 = new RandomLoadBalancer();

// Round Robin
const lb2 = new RoundRobinLoadBalancer();

// Weighted Round Robin
const lb3 = new WeightedRoundRobinLoadBalancer();

// Consistent Hash (for sticky sessions)
const lb4 = new ConsistentHashLoadBalancer();

// Least Active Connections
const lb5 = new LeastActiveLoadBalancer();
```

## Circuit Breaker

```typescript
import { CircuitBreaker } from "@dangao/bun-server";

const breaker = new CircuitBreaker({
  failureThreshold: 5,      // Open after 5 failures
  successThreshold: 3,      // Close after 3 successes
  timeout: 10000,           // Wait 10s before half-open
  volumeThreshold: 10,      // Min requests before evaluating
});

// Wrap service calls
const result = await breaker.execute(async () => {
  return orderClient.get("/api/orders/123");
});

// Check state
console.log(breaker.getState()); // "closed" | "open" | "half-open"
console.log(breaker.getStats()); // { failures, successes, ... }
```

## Rate Limiter

```typescript
import { RateLimiter } from "@dangao/bun-server";

const limiter = new RateLimiter({
  windowMs: 60000,    // 1 minute window
  max: 100,           // 100 requests per window
});

// Check if request allowed
if (limiter.tryAcquire("user-123")) {
  // Process request
} else {
  throw new Error("Rate limit exceeded");
}
```

## Retry Strategy

```typescript
import { RetryStrategyImpl } from "@dangao/bun-server";

const retry = new RetryStrategyImpl({
  maxRetries: 3,
  backoff: "exponential",
  initialDelay: 1000,
  maxDelay: 30000,
  retryOn: [500, 502, 503, 504], // Retry on these status codes
});

const result = await retry.execute(async () => {
  return orderClient.get("/api/orders/123");
});
```

## Distributed Tracing

```typescript
import {
  Tracer,
  ConsoleTraceCollector,
  MemoryTraceCollector,
} from "@dangao/bun-server";

// Setup tracer
const tracer = new Tracer({
  serviceName: "user-service",
  collector: new ConsoleTraceCollector(),
});

// Create spans
const span = tracer.startSpan("processOrder");
try {
  span.setTag("orderId", orderId);

  // Create child span
  const dbSpan = tracer.startSpan("database.query", { parent: span });
  await db.query("...");
  dbSpan.finish();

  span.finish();
} catch (error) {
  span.setStatus("error");
  span.log({ error: error.message });
  span.finish();
  throw error;
}
```

### Request Interceptors for Tracing

```typescript
import {
  TraceIdRequestInterceptor,
  RequestLogInterceptor,
  ResponseLogInterceptor,
} from "@dangao/bun-server";

const client = new ServiceClient({
  serviceName: "order-service",
  registry,
  requestInterceptors: [
    new TraceIdRequestInterceptor(tracer),
    new RequestLogInterceptor(),
  ],
  responseInterceptors: [
    new ResponseLogInterceptor(),
  ],
});
```

## Service Metrics

```typescript
import { ServiceMetricsCollector } from "@dangao/bun-server";

const metrics = new ServiceMetricsCollector({
  serviceName: "user-service",
});

// Record call metrics
metrics.recordCall("order-service", {
  success: true,
  duration: 150,
  endpoint: "/api/orders",
});

// Get metrics
const stats = metrics.getMetrics("order-service");
// { totalCalls, successRate, avgDuration, ... }
```

## Complete Microservice Setup

```typescript
import {
  Application,
  Module,
  ConfigCenterModule,
  ServiceRegistryModule,
  HealthModule,
  MetricsModule,
} from "@dangao/bun-server";

// Configure modules
ConfigCenterModule.forRoot({
  provider: "nacos",
  nacos: { serverAddr: "localhost:8848" },
});

ServiceRegistryModule.forRoot({
  provider: "nacos",
  nacos: { serverAddr: "localhost:8848" },
  service: { name: "user-service", port: 3100 },
});

HealthModule.forRoot({});
MetricsModule.forRoot({});

@Module({
  imports: [
    ConfigCenterModule,
    ServiceRegistryModule,
    HealthModule,
    MetricsModule,
  ],
})
class AppModule {}

const app = new Application({ port: 3100 });
app.registerModule(AppModule);
app.listen();
```

## Related Resources

- [Microservice Documentation](../../bun-server/docs/microservice.md)
- [Nacos Integration](../../bun-server/docs/microservice-nacos.md)
- [Service Registry](../../bun-server/docs/microservice-service-registry.md)
