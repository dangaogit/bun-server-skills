# Bun Server Cluster (ClusterManager)

`ClusterManager` spawns multiple worker processes that share a single port via SO_REUSEPORT. The master process monitors and automatically restarts crashed workers.

> **Platform note**: SO_REUSEPORT (`reusePort: true`) works on Linux. On macOS and Windows it is silently ignored — only one worker will bind the port.

## ClusterManager API

```typescript
import { ClusterManager } from '@dangao/bun-server';

// Create manager (runs in master process)
const manager = new ClusterManager({
  workers: 'auto' | number,  // 'auto' = number of CPU cores
  scriptPath: import.meta.path,
  port: 3000,
  hostname?: string,
});

manager.start();         // Spawn workers
await manager.stop();    // Graceful shutdown
```

## Static Methods

```typescript
// Returns true in worker processes, false in master
ClusterManager.isWorker(): boolean

// Returns worker index (0, 1, ...) in workers; -1 in master
ClusterManager.getWorkerId(): number
```

## Master / Worker Pattern

```typescript
import { Application, ClusterManager, Module, Controller, GET } from '@dangao/bun-server';

const PORT = Number(process.env.PORT ?? 3000);

if (!ClusterManager.isWorker()) {
  // Master: spawn workers and handle signals
  const manager = new ClusterManager({
    workers: process.env.WORKERS ? Number(process.env.WORKERS) : 'auto',
    scriptPath: import.meta.path,
    port: PORT,
  });

  manager.start();

  process.on('SIGINT', async () => {
    await manager.stop();
    process.exit(0);
  });
} else {
  // Worker: run the application
  @Controller('/api')
  class AppController {
    @GET('/ping')
    public ping(): object {
      return { pong: true, worker: ClusterManager.getWorkerId() };
    }
  }

  @Module({ controllers: [AppController] })
  class WorkerModule {}

  const app = new Application({
    port: PORT,
    reusePort: true,           // Required for port sharing
    enableSignalHandlers: true,
  });
  app.registerModule(WorkerModule);
  await app.listen();
}
```

## Environment Variable Configuration

```bash
# Number of workers (default: auto = CPU cores)
WORKERS=4 bun run src/main.ts

# Custom port
PORT=8080 bun run src/main.ts
```

## When to Use

- High-throughput APIs that need to utilize all CPU cores.
- Long-lived processes where worker crash recovery is important.
- Linux production deployments (SO_REUSEPORT provides best performance).

## Limitations

- `reusePort: true` is Bun-exclusive and Linux-only; silently ignored on macOS/Windows.
- On non-Linux systems, multiple workers start but only one can accept connections.
- Cluster mode on Node.js uses `node:child_process` and may behave differently from Bun's cluster.
