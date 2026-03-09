# Bun Server Dashboard Module

## Overview

`DashboardModule` provides an embedded monitoring Web UI at `/_dashboard` (configurable). It shows application health, all registered routes, system information, and supports Markdown rendering — all with zero external dependencies.

Requires Bun >= `1.3.10`.

## Setup

```typescript
import { Module, DashboardModule } from "@dangao/bun-server";

DashboardModule.forRoot({
  path: "/_dashboard",  // Default
});

@Module({
  imports: [DashboardModule],
})
class AppModule {}
```

### With Basic Auth

Protect the dashboard with HTTP Basic Authentication:

```typescript
DashboardModule.forRoot({
  path: "/_dashboard",
  auth: {
    username: "admin",
    password: "secret",
  },
});
```

### With HealthModule Integration

When `HealthModule` is registered alongside `DashboardModule`, the `/_dashboard/api/health` endpoint automatically runs all registered health indicators:

```typescript
import { HealthModule, DatabaseHealthIndicator } from "@dangao/bun-server";

HealthModule.forRoot({
  path: "/health",
  indicators: [DatabaseHealthIndicator],
});

DashboardModule.forRoot({});

@Module({
  imports: [HealthModule, DashboardModule, DatabaseModule],
})
class AppModule {}
```

## Available Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/_dashboard` | Dashboard HTML UI |
| `GET` | `/_dashboard/api/system` | System info (uptime, memory, platform, Bun version) |
| `GET` | `/_dashboard/api/routes` | All registered routes (method, path, controller, methodName) |
| `GET` | `/_dashboard/api/health` | Health status (integrates with HealthModule if registered) |
| `POST` | `/_dashboard/api/markdown` | Render Markdown to HTML (powered by Bun built-in Markdown parser) |

### System Info Response

```json
// GET /_dashboard/api/system
{
  "uptime": 3600,
  "memory": {
    "rss": 52428800,
    "heapUsed": 20971520,
    "heapTotal": 41943040
  },
  "platform": "linux",
  "bunVersion": "1.3.11"
}
```

### Routes Response

```json
// GET /_dashboard/api/routes
[
  { "method": "GET", "path": "/users", "controller": "UserController", "methodName": "list" },
  { "method": "POST", "path": "/users", "controller": "UserController", "methodName": "create" }
]
```

### Markdown Render

```bash
# POST /_dashboard/api/markdown
curl -X POST http://localhost:3000/_dashboard/api/markdown \
  -H "Content-Type: application/json" \
  -d '{"content": "# Hello\n\nThis is **markdown**."}'

# Response: { "html": "<h1>Hello</h1><p>This is <strong>markdown</strong>.</p>" }
```

## Notes

- `DashboardModule` does not require `forRoot()` options — calling `DashboardModule.forRoot()` with no arguments uses all defaults.
- Dashboard is intended for development and internal monitoring. In production, use `auth` to restrict access.
- Markdown rendering uses `Bun.markdown.html()` (GFM-compatible; introduced in Bun 1.3.8+).

## Related Resources

- [Health & Metrics](health-metrics.md)
- [Debug Module](debug.md)
