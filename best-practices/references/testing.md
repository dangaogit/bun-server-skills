# Bun Server Testing

## Overview

The testing utilities provide an isolated module environment for integration testing, HTTP-level testing, and performance testing. Works with `bun:test` (built-in test runner).

## Integration Testing with Test.createTestingModule

### Basic Setup

```typescript
import { Test } from "@dangao/bun-server";
import { describe, it, expect } from "bun:test";

describe("UserService", () => {
  it("should return users", async () => {
    const moduleRef = await Test.createTestingModule({
      providers: [UserService, DatabaseService],
    }).compile();

    const service = moduleRef.get(UserService);
    const users = await service.getUsers();
    expect(users).toBeArray();
  });
});
```

### Mocking Dependencies with overrideProvider

```typescript
const mockDatabaseService = {
  query: async () => [{ id: 1, name: "Alice" }],
};

const moduleRef = await Test.createTestingModule({
  providers: [UserService, DatabaseService],
})
  .overrideProvider(DatabaseService).useValue(mockDatabaseService)
  .compile();

const service = moduleRef.get(UserService);
```

Override methods:

```typescript
builder
  .overrideProvider(SomeService).useValue(mockValue)       // Fixed value
  .overrideProvider(SomeService).useClass(MockSomeService) // Alternative class
  .overrideProvider(SomeService).useFactory(() => mock)    // Factory function
```

### Using Symbol Tokens

```typescript
const moduleRef = await Test.createTestingModule({
  providers: [UserService],
})
  .overrideProvider(DATABASE_SERVICE_TOKEN).useValue(mockDb)
  .compile();

const service = moduleRef.get(UserService);
```

### Importing Full Modules

```typescript
const moduleRef = await Test.createTestingModule({
  imports: [AppModule],
})
  .overrideProvider(EMAIL_SERVICE_TOKEN).useValue(mockEmailService)
  .compile();
```

## HTTP Integration Testing with TestHttpClient

`TestHttpClient` starts a real HTTP server on a random port for end-to-end testing:

```typescript
describe("UserController", () => {
  it("GET /users returns 200", async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [UserModule],
    })
      .overrideProvider(UserService).useValue(mockUserService)
      .compile();

    const client = await moduleRef.createHttpClient();

    try {
      const res = await client.get("/users");
      expect(res.status).toBe(200);
      expect(res.ok).toBe(true);
      expect(res.body).toBeArray();
    } finally {
      await client.close(); // Always close to avoid port leaks
    }
  });

  it("POST /users creates user", async () => {
    const client = await moduleRef.createHttpClient();

    const res = await client.post("/users", {
      body: { name: "Bob", email: "bob@example.com" },
    });

    expect(res.status).toBe(201);
    expect((res.body as any).name).toBe("Bob");

    await client.close();
  });
});
```

### TestHttpClient Methods

```typescript
// All methods accept optional RequestOptions
interface RequestOptions {
  headers?: Record<string, string>;
  body?: unknown;            // Serialized to JSON automatically
  query?: Record<string, string>;
}

// Response shape
interface TestResponse {
  status: number;
  headers: Headers;
  body: unknown;   // Parsed JSON or raw string
  text: string;    // Raw response text
  ok: boolean;     // status >= 200 && < 300
}

// Available methods
client.get(path, options?)
client.post(path, options?)
client.put(path, options?)
client.delete(path, options?)
client.patch(path, options?)
client.close()  // Stop the test server
```

## Performance Testing

### PerformanceHarness — Benchmark

Measure the operations-per-second of any synchronous or async function:

```typescript
import { PerformanceHarness } from "@dangao/bun-server";
import { describe, it, expect } from "bun:test";

describe("performance", () => {
  it("JSON parsing benchmark", async () => {
    const result = await PerformanceHarness.benchmark(
      "parse JSON",
      10_000,
      (i) => {
        JSON.parse('{"key": "value", "n": ' + i + "}");
      },
    );

    console.log(`${result.name}: ${result.opsPerSecond.toFixed(0)} ops/sec`);
    expect(result.opsPerSecond).toBeGreaterThan(100_000);
  });
});
```

`BenchmarkResult` shape:

```typescript
interface BenchmarkResult {
  name: string;
  iterations: number;
  durationMs: number;
  opsPerSecond: number;
}
```

### StressTester — Concurrent Load Test

Run many concurrent async tasks and collect error statistics:

```typescript
import { StressTester } from "@dangao/bun-server";

const result = await StressTester.run(
  "concurrent HTTP requests",
  1000,   // Total iterations
  50,     // Concurrency
  async (i) => {
    const res = await fetch(`http://localhost:3000/users/${i % 10}`);
    if (!res.ok) throw new Error(`Request ${i} failed: ${res.status}`);
  },
);

console.log(`Completed ${result.iterations} iterations in ${result.durationMs.toFixed(0)}ms`);
console.log(`Errors: ${result.errors}`);
expect(result.errors).toBe(0);
```

`StressResult` shape:

```typescript
interface StressResult {
  name: string;
  iterations: number;
  concurrency: number;  // Actual concurrency (min of specified and iterations)
  durationMs: number;
  errors: number;       // Count of tasks that threw an error
}
```

## ApplicationOptions for Tests

When creating the application in tests, signal handlers are automatically disabled to prevent test runner interference:

```typescript
// TestingModule.createApplication() sets enableSignalHandlers: false by default
const client = await moduleRef.createHttpClient({
  // Additional ApplicationOptions can be passed here
});
```

## Related Resources

- [Client Generation](client.md)
- [Dependency Injection](dependency-injection.md)
- [Module System](module-system.md)
