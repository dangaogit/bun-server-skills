# Bun Server Platform Adapter (v3.0+)

v3.0 introduces a **Platform Adapter Layer** that enables running the same codebase on both **Bun** and **Node.js 22+** with no application-level changes.

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                  Application Layer                    │
│   Controllers / Services / Modules / Middleware       │
└──────────────────────┬───────────────────────────────┘
                       │ getRuntime()
┌──────────────────────▼───────────────────────────────┐
│              Platform Adapter Layer                   │
│  IFsAdapter · ICryptoAdapter · IParserAdapter         │
│  IProcessAdapter · IHttpDriver · IWebSocket           │
└──────┬───────────────────────────────┬───────────────┘
       │                               │
┌──────▼──────┐                 ┌──────▼──────┐
│ BunPlatform │                 │ NodePlatform│
│ Bun.serve   │                 │ node:http   │
│ Bun.file    │                 │ node:fs     │
│ Bun.Crypto  │                 │ node:crypto │
└─────────────┘                 └─────────────┘
```

## Adapter Interfaces

| Interface | Bun Implementation | Node.js Implementation |
|---|---|---|
| `IFsAdapter` | `Bun.file`, `Bun.write`, `Bun.Glob` | `node:fs/promises` |
| `ICryptoAdapter` | `Bun.CryptoHasher` | `node:crypto` HMAC/hash |
| `IParserAdapter` | `Bun.JSONC`, `Bun.JSON5`, `Bun.JSONL`, `Bun.markdown` | `jsonc-parser`, `json5`, `marked` |
| `IProcessAdapter` | `spawn` (bun), `Bun.sleep` | `node:child_process`, `setTimeout` |
| `IHttpDriver` | `Bun.serve` | `node:http.createServer` |
| `IWebSocket<T>` | `Bun.ServerWebSocket<T>` | `ws` package |

## Runtime Detection Priority

```
1. Bootstrap config  →  new Application({ platform: 'node' })
2. CLI argument      →  --platform=node
3. Environment var   →  BUN_SERVER_PLATFORM=node
4. Auto-detect       →  typeof Bun !== 'undefined' ? 'bun' : 'node'
```

## Platform Configuration

```typescript
import { Application } from '@dangao/bun-server';

// Option 1: Code (highest priority)
const app = new Application({ platform: 'node' }); // 'bun' | 'node'

// Option 2: CLI argument
// bun run src/main.ts --platform=node
// node dist/main.js  (auto-detected)

// Option 3: Environment variable
// BUN_SERVER_PLATFORM=node node dist/main.js
```

## Support Matrix

| Feature | Bun | Node.js 22+ |
|---|---|---|
| HTTP server | `Bun.serve` (native) | `node:http` |
| WebSocket | `Bun.ServerWebSocket` | `ws` package |
| File I/O | `Bun.file / Bun.write` | `node:fs/promises` |
| Crypto / JWT | `Bun.CryptoHasher` | `node:crypto` |
| JSONC / JSON5 parsing | `Bun.JSONC`, `Bun.JSON5` | `jsonc-parser`, `json5` |
| SQLite | `bun:sqlite` | `better-sqlite3` |
| PostgreSQL | `Bun.SQL` | `postgres` package |
| MySQL | `Bun.SQL` | `mysql2` package |
| `idleTimeout` | Yes | Silently ignored |
| `reusePort` | Yes | Silently ignored |
| SSE TCP keepalive | Yes | Heartbeat injection only |

## Database Auto-Adaptation

`DatabaseModule` automatically selects drivers — no extra configuration needed:

```typescript
// Same config works on both runtimes
DatabaseModule.forRoot({
  connections: [
    {
      name: 'default',
      type: 'sqlite',
      database: './data/app.db',
    },
    {
      name: 'pg',
      type: 'postgres',
      url: process.env.DATABASE_URL,
    },
  ],
})
```

## v3 Breaking API Changes

### `app.getServer()`

```typescript
// Before (v2)
import type { } from 'bun';
const server: Bun.Server | undefined = app.getServer();

// After (v3)
import type { IServerHandle } from '@dangao/bun-server';
const handle: IServerHandle | undefined = app.getServer();
handle?.port;      // number
handle?.hostname;  // string
await handle?.stop();

// Raw native access (not recommended — type is unknown)
const native: unknown = app.getNativeServer();
```

### WebSocket client type

```typescript
// Before (v2)
import type { ServerWebSocket } from 'bun';
handleOpen(ws: ServerWebSocket<unknown>) {}

// After (v3)
import type { IWebSocket } from '@dangao/bun-server';
handleOpen(ws: IWebSocket<unknown>) {}
```

## Bun-Exclusive Features

These options in `ApplicationOptions` are silently ignored on Node.js:

| Option | Effect on Bun |
|---|---|
| `idleTimeout` | TCP idle timeout via `Bun.serve` |
| `reusePort` | Port reuse via `Bun.serve` |
| `sseKeepAlive` | Uses `server.timeout(req, 0)` for TCP keep-alive |

## Node.js Startup Guide

```bash
# Install (peer dependencies included automatically)
npm install @dangao/bun-server

# Build (compile TypeScript to JS targeting Node.js)
bun build src/main.ts --target=node --outdir=dist
# Or: npx tsc

# Run
node dist/main.js   # Platform auto-detected as 'node'
```
