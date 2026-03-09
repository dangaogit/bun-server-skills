# Bun Server Debug Module

## Overview

`DebugModule` records all HTTP requests in a ring buffer and provides an embedded Debug UI at `/_debug`. Useful during development for inspecting request/response details, timing, and replaying requests.

Requires Bun >= `1.3.10` (the `Bun.JSONL.parse()` capability used here was introduced in Bun 1.3.7+).

## Setup

```typescript
import { Module, DebugModule } from "@dangao/bun-server";

DebugModule.forRoot({
  enabled: true,       // Default: true
  maxRecords: 500,     // Ring buffer size (default: 500)
  recordBody: true,    // Record request body (default: true)
  path: "/_debug",     // Debug UI path (default: "/_debug")
});

@Module({
  imports: [DebugModule],
})
class AppModule {}
```

When `enabled: false`, no middleware is registered and no requests are recorded.

## Debug UI

Navigate to `http://localhost:3000/_debug` to view the debug UI. It shows:
- All recorded requests (newest first)
- Request details: method, path, headers, body, response status
- Timing information (total duration in ms)
- Matched route and controller metadata

## Accessing the Recorder Programmatically

Inject `RequestRecorder` via the `DEBUG_RECORDER_TOKEN` to access recordings from your own code:

```typescript
import {
  Injectable,
  Inject,
  RequestRecorder,
  DEBUG_RECORDER_TOKEN,
} from "@dangao/bun-server";

@Injectable()
class DiagnosticsService {
  constructor(
    @Inject(DEBUG_RECORDER_TOKEN) private readonly recorder: RequestRecorder
  ) {}

  // Get all recorded requests (newest first)
  getAll() {
    return this.recorder.getAll();
  }

  // Lookup a specific request by ID
  getById(id: string) {
    return this.recorder.getById(id);
  }

  // Current number of records in the buffer
  count() {
    return this.recorder.getCount();
  }

  // Clear all records
  clear() {
    this.recorder.clear();
  }

  // Export to JSONL format (one JSON object per line)
  exportJsonl(): string {
    return this.recorder.exportToJsonl();
  }

  // Import from a JSONL string
  importJsonl(content: string): void {
    this.recorder.importFromJsonl(content);
  }
}
```

## RequestRecord Structure

Each recorded request conforms to:

```typescript
interface RequestRecord {
  id: string;           // Unique ID (hex timestamp + counter)
  timestamp: number;    // Unix timestamp (ms)
  request: {
    method: string;     // HTTP method
    path: string;       // Request path
    headers: Record<string, string>;
    body?: unknown;     // Recorded if recordBody: true
  };
  response: {
    status: number;     // HTTP status code
    headers: Record<string, string>;
    bodySize: number;   // Response body size in bytes
  };
  timing: {
    total: number;      // Total request duration in ms
  };
  metadata: {
    matchedRoute?: string;   // Matched route pattern
    controller?: string;     // Controller class name
    methodName?: string;     // Handler method name
  };
}
```

## JSONL Export / Import

Use JSONL export for saving request traces to disk or sharing with teammates:

```typescript
// Export
const jsonl = recorder.exportToJsonl();
await Bun.write("./traces/requests.jsonl", jsonl);

// Import (useful for replaying or analyzing in tests)
const content = await Bun.file("./traces/requests.jsonl").text();
recorder.importFromJsonl(content);

// Static parse without importing into recorder
const records = RequestRecorder.parseJsonl(content);
```

## Common Pattern: Disable in Production

```typescript
DebugModule.forRoot({
  enabled: process.env.NODE_ENV !== "production",
  maxRecords: 200,
  recordBody: false,  // Reduce memory usage
});
```

## Related Resources

- [Dashboard Module](dashboard.md)
- [Middleware](middleware.md)
