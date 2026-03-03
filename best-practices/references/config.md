# Bun Server Configuration Module

## Setup ConfigModule

### Synchronous (forRoot)

```typescript
import { Module, ConfigModule } from "@dangao/bun-server";

ConfigModule.forRoot({
  defaultConfig: {
    port: 3000,
    db: { host: "localhost", port: 5432 },
  },
  load: (env) => ({
    port: Number(env.PORT) || 3000,
    db: {
      host: env.DB_HOST ?? "localhost",
      port: Number(env.DB_PORT) || 5432,
    },
  }),
  validate: (config) => {
    if (!config.db?.host) throw new Error("DB_HOST is required");
  },
});

@Module({
  imports: [ConfigModule],
})
class AppModule {}
```

Priority order (highest to lowest): `load()` env result > `defaultConfig`.

### Asynchronous (forRootAsync)

Use `forRootAsync` when you need to load configuration from files:

```typescript
ConfigModule.forRootAsync({
  useFactory: async () => ({
    configFiles: ["./config/app.json", "./config/app.jsonc"],  // Loaded in order, later files override
    load: (env) => ({
      port: Number(env.PORT) || 3000,
    }),
  }),
});
```

Supported file formats: `.json`, `.jsonc` (Bun 1.3.6+), `.json5` (Bun 1.3.7+).

Priority order: `load()` env result > `configFiles` > `defaultConfig`.

## Inject ConfigService

```typescript
import {
  Injectable,
  Inject,
  ConfigService,
  CONFIG_SERVICE_TOKEN,
} from "@dangao/bun-server";

@Injectable()
class AppService {
  constructor(
    @Inject(CONFIG_SERVICE_TOKEN) private readonly config: ConfigService
  ) {}

  getPort(): number {
    return this.config.get<number>("port", 3000)!;
  }

  getDbHost(): string {
    return this.config.getRequired<string>("db.host"); // Throws if not found
  }
}
```

## ConfigService API

### get — Dot-path access with default

```typescript
// Access nested values with dot notation
const host = config.get<string>("db.host", "localhost");
const port = config.get<number>("db.port", 5432);
const name = config.get<string>("app.name");  // undefined if not found

// Get full config object
const all = config.getAll();
```

### getRequired — Throw if missing

```typescript
// Throws Error if key is not found
const secret = config.getRequired<string>("jwt.secret");
```

### withNamespace — Scoped view

```typescript
const dbConfig = config.withNamespace("db");
const host = dbConfig.get<string>("host"); // Reads "db.host" from the full config
const port = dbConfig.get<number>("port"); // Reads "db.port"
```

### Dynamic Updates

```typescript
// Listen for config changes (e.g., from config center)
const unsubscribe = config.onConfigUpdate((newConfig) => {
  console.log("Config updated:", newConfig);
});

// Stop listening
unsubscribe();

// Manual update / merge
config.updateConfig({ ...newFullConfig });
config.mergeConfig({ port: 4000 }); // Partial update
```

## Namespace Option

Use `namespace` to logically group a module's config:

```typescript
ConfigModule.forRoot({
  defaultConfig: { app: { name: "My App" } },
  namespace: "app",  // All get() calls are prefixed with "app."
});

// In service:
config.get("name");     // Reads "app.name"
config.get("app.name"); // Also works (prefix is not duplicated)
```

## Config Center Integration (Nacos)

Integrate with Nacos for distributed configuration. Requires `ConfigCenterModule` to be registered first:

```typescript
import { ConfigCenterModule } from "@dangao/bun-server";

ConfigModule.forRoot({
  defaultConfig: { port: 3000 },
  configCenter: {
    enabled: true,
    configCenterPriority: true,   // Config center overrides local config (default: true)
    configs: new Map([
      // key: dot-path in local config, value: Nacos dataId/group
      ["database", { dataId: "db-config", groupName: "DEFAULT_GROUP" }],
      ["app.name", { dataId: "app-name", groupName: "APP_GROUP" }],
    ]),
  },
});

@Module({
  imports: [
    ConfigCenterModule.forRoot({ /* Nacos options */ }),
    ConfigModule,
  ],
})
class AppModule {}
```

Config center values are loaded and watchers are set up automatically at `app.listen()`.

## Common Patterns

### Typed Config Interface

```typescript
interface AppConfig {
  port: number;
  db: { host: string; port: number; name: string };
  jwt: { secret: string; expiresIn: string };
}

ConfigModule.forRoot<AppConfig>({
  defaultConfig: {
    port: 3000,
    db: { host: "localhost", port: 5432, name: "mydb" },
    jwt: { secret: "change-me", expiresIn: "1h" },
  },
  load: (env) => ({
    db: { host: env.DB_HOST ?? "localhost" },
    jwt: { secret: env.JWT_SECRET ?? "change-me" },
  }),
});

// Type-safe injection
@Injectable()
class DatabaseService {
  constructor(
    @Inject(CONFIG_SERVICE_TOKEN) private config: ConfigService<AppConfig>
  ) {}

  connect() {
    const host = this.config.getRequired<string>("db.host");
    const port = this.config.get<number>("db.port", 5432)!;
    // ...
  }
}
```

## Related Resources

- [Module System](module-system.md)
- [Microservice / Config Center](microservice.md)
