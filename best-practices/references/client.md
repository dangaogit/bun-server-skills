# Bun Server Client Generation

## Overview

The client generation utilities allow you to expose a type-safe route manifest from your server and consume it with a generated client on the browser or another service — without writing any fetch boilerplate.

## Server Side: Expose the Route Manifest

Use `ClientGenerator.generate()` to extract all registered routes and expose them via a controller endpoint:

```typescript
import {
  Controller,
  GET,
  ClientGenerator,
} from "@dangao/bun-server";

@Controller("/manifest")
class ManifestController {
  @GET("/")
  getManifest() {
    return ClientGenerator.generate();
  }
}
```

`ClientGenerator.generate()` returns a `RouteManifest`:

```typescript
// Example response from GET /manifest
{
  "routes": [
    { "method": "GET",  "path": "/users",     "controllerName": "UserController",  "methodName": "list" },
    { "method": "POST", "path": "/users",     "controllerName": "UserController",  "methodName": "create" },
    { "method": "GET",  "path": "/users/:id", "controllerName": "UserController",  "methodName": "getUser" }
  ]
}
```

You can also generate a formatted JSON string for file output:

```typescript
const json = ClientGenerator.generateJSON();
await Bun.write("./client/manifest.json", json);
```

## Client Side: Create the API Client

Fetch the manifest and pass it to `createClient()`:

```typescript
import { createClient } from "@dangao/bun-server";

const manifest = await fetch("http://localhost:3000/manifest").then(r => r.json());

const client = createClient(manifest, {
  baseUrl: "http://localhost:3000",
  headers: {
    Authorization: "Bearer my-token",
  },
});
```

## Calling API Methods

The client is organized by controller name. The controller name has its `Controller` suffix stripped and is lowercased:
- `UserController` → `client.user`
- `OrderController` → `client.order`

Each method on the controller maps directly to a function:

```typescript
// GET /users
const users = await client.user.list();

// GET /users/:id — path params
const user = await client.user.getUser({ params: { id: "42" } });

// POST /users — request body
const created = await client.user.create({
  body: { name: "Alice", email: "alice@example.com" },
});

// GET /users?page=2&limit=10 — query params
const page2 = await client.user.list({ query: { page: "2", limit: "10" } });

// Custom per-request headers
const result = await client.user.getUser({
  params: { id: "1" },
  headers: { "X-Trace-Id": "abc123" },
});
```

### ClientRequestOptions

```typescript
interface ClientRequestOptions {
  params?: Record<string, string>;   // Replaces :param placeholders in path
  query?: Record<string, string>;    // Appended as URL query string
  body?: unknown;                    // Serialized to JSON (ignored for GET)
  headers?: Record<string, string>;  // Per-request headers (merge with global)
}
```

## Custom fetch for Testing

Inject a custom `fetch` implementation to mock HTTP calls in tests:

```typescript
import { createClient } from "@dangao/bun-server";
import { describe, it, expect, mock } from "bun:test";

const mockFetch = mock(async (url: string, options: RequestInit) => {
  return new Response(JSON.stringify([{ id: 1 }]), {
    status: 200,
    headers: { "Content-Type": "application/json" },
  });
});

const client = createClient(manifest, {
  baseUrl: "http://localhost:3000",
  fetch: mockFetch as typeof fetch,
});

const users = await client.user.list();
expect(users).toEqual([{ id: 1 }]);
```

## Types Reference

```typescript
interface RouteManifest {
  routes: RouteManifestEntry[];
}

interface RouteManifestEntry {
  method: string;
  path: string;
  controllerName: string;
  methodName: string;
}

interface ClientConfig {
  baseUrl: string;
  headers?: Record<string, string>;
  fetch?: typeof fetch;  // Custom fetch for testing
}
```

## Related Resources

- [Controllers and Routing](controller-routing.md)
- [Testing](testing.md)
