# Async Module Configuration (forRootAsync)

Some modules support `forRootAsync()` for configuration that depends on asynchronous sources — remote config services, secrets managers, or database lookups. The async factory runs during `Application.listen()` before the server starts.

## AsyncModuleOptions Interface

```typescript
interface AsyncModuleOptions<T> {
  imports?: ModuleClass[];         // Modules whose providers are needed in useFactory
  inject?: ProviderToken[];        // Tokens to inject into useFactory (in order)
  useFactory: (...deps: unknown[]) => T | Promise<T>;
}
```

## Supported Modules

| Module | Method |
|---|---|
| `ConfigModule` | `ConfigModule.forRootAsync(options)` |
| `DatabaseModule` | `DatabaseModule.forRootAsync(options)` |
| `CacheModule` | `CacheModule.forRootAsync(options)` |

## ConfigModule.forRootAsync

Load configuration from a remote source before the server starts:

```typescript
import { Module, ConfigModule } from '@dangao/bun-server';

@Module({
  imports: [
    ConfigModule.forRootAsync({
      useFactory: async () => {
        const remoteConfig = await loadRemoteConfig(); // fetch from Vault, AWS SSM, etc.
        return {
          defaultConfig: remoteConfig,
        };
      },
    }),
  ],
  controllers: [AppController],
})
class AppModule {}
```

## DatabaseModule.forRootAsync with ConfigService

Inject `ConfigService` to derive database options from the config:

```typescript
import {
  Module,
  ConfigModule,
  DatabaseModule,
  CONFIG_SERVICE_TOKEN,
} from '@dangao/bun-server';
import type { ConfigService } from '@dangao/bun-server';

ConfigModule.forRoot({
  defaultConfig: {
    db: { url: process.env.DATABASE_URL ?? 'postgres://localhost/mydb' },
  },
});

@Module({
  imports: [
    ConfigModule,
    DatabaseModule.forRootAsync({
      imports: [ConfigModule],          // Make ConfigModule's providers available
      inject: [CONFIG_SERVICE_TOKEN],   // Inject ConfigService as first arg
      useFactory: (config: ConfigService) => ({
        connections: [
          {
            name: 'default',
            type: 'postgres',
            url: config.get<string>('db.url'),
          },
        ],
      }),
    }),
  ],
})
class AppModule {}
```

## CacheModule.forRootAsync

```typescript
import {
  Module,
  ConfigModule,
  CacheModule,
  CONFIG_SERVICE_TOKEN,
} from '@dangao/bun-server';
import type { ConfigService } from '@dangao/bun-server';

@Module({
  imports: [
    ConfigModule.forRoot({ defaultConfig: { cache: { ttl: 300, host: 'localhost' } } }),
    CacheModule.forRootAsync({
      imports: [ConfigModule],
      inject: [CONFIG_SERVICE_TOKEN],
      useFactory: (config: ConfigService) => ({
        ttl: config.get<number>('cache.ttl', 60),
        store: 'redis',
        host: config.get<string>('cache.host', 'localhost'),
        port: 6379,
      }),
    }),
  ],
})
class AppModule {}
```

## Key Rules

- `useFactory` dependencies are resolved in the order listed in `inject`.
- Async providers are initialized **sequentially** during `listen()`.
- Always include required module in both `imports` and `inject` when pulling from another module's provider.
- Use `forRoot()` (synchronous) when config is already available at module definition time; use `forRootAsync()` only when you need to await external data.
