# Bun Server Lifecycle Hooks

Lifecycle hooks let `@Injectable` services and `@Controller` classes participate in the application's startup and shutdown sequence.

## Interfaces

| Interface | Method | When Called |
|---|---|---|
| `ComponentClassBeforeCreate` | `static onBeforeCreate()` | Right before the component instance is created |
| `OnAfterCreate` | `onAfterCreate()` | Right after instance creation and post-processors |
| `OnModuleInit` | `onModuleInit()` | After all module providers are registered |
| `OnModuleDestroy` | `onModuleDestroy()` | During shutdown (reverse order) |
| `OnBeforeDestroy` | `onBeforeDestroy()` | Before `onModuleDestroy` during shutdown |
| `OnAfterDestroy` | `onAfterDestroy()` | After `onModuleDestroy` during shutdown |
| `OnApplicationBootstrap` | `onApplicationBootstrap()` | After all modules init, before server listens |
| `OnApplicationShutdown` | `onApplicationShutdown(signal?)` | When graceful shutdown begins |

## Execution Order

**Creation (per component)**:
`onBeforeCreate` (static) → instantiate → `onAfterCreate`

**Startup**:
`onModuleInit` (all modules) → `onApplicationBootstrap` (all modules) → server starts

**Shutdown** (reverse registration order):
`onApplicationShutdown` → `onBeforeDestroy` → `onModuleDestroy` → `onAfterDestroy`

> For `Scope.SCOPED` components, destroy hooks run automatically at the end of each request context.

## Provider Deduplication

When the same provider instance is registered/exported by multiple tokens, `onModuleInit` runs **only once** for that instance — no duplicate initialization side effects.

## Component Lifecycle Example

```typescript
import {
  Injectable,
  OnAfterCreate,
  OnModuleInit,
  OnBeforeDestroy,
  OnModuleDestroy,
  OnAfterDestroy,
} from '@dangao/bun-server';
import type { ComponentClassBeforeCreate } from '@dangao/bun-server';

@Injectable()
class DatabaseService
  implements
    OnAfterCreate,
    OnModuleInit,
    OnBeforeDestroy,
    OnModuleDestroy,
    OnAfterDestroy
{
  private connected = false;

  public static onBeforeCreate(): void {
    console.log('[DatabaseService] Before create');
  }

  public onAfterCreate(): void {
    console.log('[DatabaseService] After create');
  }

  public async onModuleInit(): Promise<void> {
    console.log('[DatabaseService] Connecting...');
    await this.connect();
    this.connected = true;
  }

  public async onModuleDestroy(): Promise<void> {
    console.log('[DatabaseService] Closing connection...');
    await this.disconnect();
    this.connected = false;
  }

  public onBeforeDestroy(): void {
    console.log('[DatabaseService] Before destroy');
  }

  public onAfterDestroy(): void {
    console.log('[DatabaseService] After destroy');
  }

  public isConnected(): boolean {
    return this.connected;
  }

  private async connect(): Promise<void> { /* Open DB connection */ }
  private async disconnect(): Promise<void> { /* Close DB connection */ }
}
```

## Application-Level Hooks

```typescript
import {
  Injectable,
  OnApplicationBootstrap,
  OnApplicationShutdown,
} from '@dangao/bun-server';

@Injectable()
class AppService implements OnApplicationBootstrap, OnApplicationShutdown {
  public onApplicationBootstrap(): void {
    console.log('Server is ready and listening');
  }

  public onApplicationShutdown(signal?: string): void {
    console.log(`Shutting down (signal: ${signal ?? 'none'})`);
  }
}
```

## Controller Hooks

Controllers support the same lifecycle interfaces:

```typescript
import { Controller, GET, OnAfterCreate, OnBeforeDestroy } from '@dangao/bun-server';

@Controller('/health')
class HealthController implements OnAfterCreate, OnBeforeDestroy {
  public static onBeforeCreate(): void {
    console.log('[HealthController] Before create');
  }

  public onAfterCreate(): void {
    console.log('[HealthController] After create');
  }

  public onBeforeDestroy(): void {
    console.log('[HealthController] Cleaning up...');
  }

  @GET('/')
  public get(): object {
    return { ok: true };
  }
}
```

## Complete Module with Lifecycle

```typescript
import {
  Module,
  Application,
  OnModuleInit,
  OnApplicationBootstrap,
  OnApplicationShutdown,
} from '@dangao/bun-server';

@Injectable()
class CacheService implements OnModuleInit, OnApplicationShutdown {
  public async onModuleInit(): Promise<void> {
    await this.connect();
  }

  public async onApplicationShutdown(): Promise<void> {
    await this.flush();
    await this.disconnect();
  }

  private async connect() { /* ... */ }
  private async flush() { /* ... */ }
  private async disconnect() { /* ... */ }
}

@Module({ providers: [CacheService] })
class AppModule {}

const app = new Application({ port: 3000 });
app.registerModule(AppModule);
await app.listen(); // onModuleInit fires before server starts
```

## Best Practices

- Use `onModuleInit` for async resource setup (DB connections, cache warming).
- Use `onApplicationBootstrap` for work that needs all modules ready (seed data, background jobs).
- Use `onApplicationShutdown` + `onModuleDestroy` for graceful release of external resources.
- Async hooks (`Promise<void>`) are fully supported; the framework `await`s them in sequence.
