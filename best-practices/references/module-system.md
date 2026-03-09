# Bun Server Module System

## Basic Module

```typescript
import { Module, Controller, Injectable } from "@dangao/bun-server";

@Injectable()
class UserService {
  findAll() {
    return [{ id: 1, name: "Alice" }];
  }
}

@Controller("/users")
class UserController {
  constructor(private readonly userService: UserService) {}
}

@Module({
  controllers: [UserController],
  providers: [UserService],
})
class UserModule {}
```

## Module Structure

```typescript
@Module({
  imports: [],      // Import other modules
  controllers: [],  // Controllers in this module
  providers: [],    // Service providers
  exports: [],      // Services exported for other modules
})
class SomeModule {}
```

## Module Imports and Exports

```typescript
// user.module.ts
@Module({
  providers: [UserService],
  exports: [UserService], // Export for other modules
})
class UserModule {}

// order.module.ts
@Module({
  imports: [UserModule], // Import UserModule
  providers: [OrderService],
  controllers: [OrderController],
})
class OrderModule {}

// order.service.ts
@Injectable()
class OrderService {
  constructor(private readonly userService: UserService) {} // Can inject
}
```

## forRoot Pattern

Used to configure global modules (called once):

```typescript
import {
  ConfigModule,
  EventModule,
  LoggerModule,
  SecurityModule,
  LogLevel,
} from "@dangao/bun-server";

// Configure global modules
ConfigModule.forRoot({
  defaultConfig: {
    app: { name: "My App", port: 3100 },
    database: { url: "postgres://..." },
  },
});

EventModule.forRoot({
  wildcard: true,
  maxListeners: 20,
});

LoggerModule.forRoot({
  level: LogLevel.INFO,
});

// Import in root module
@Module({
  imports: [ConfigModule, EventModule, LoggerModule],
  controllers: [AppController],
})
class AppModule {}
```

**Important**: `forRoot()` must be called before module definitions.

## Built-in Modules Requiring forRoot

| Module | Purpose | Configuration Example |
|--------|---------|----------------------|
| `ConfigModule` | Configuration management | `forRoot({ defaultConfig: {} })` |
| `EventModule` | Event system | `forRoot({ wildcard: true })` |
| `LoggerModule` | Logging | `forRoot({ level: LogLevel.INFO })` |
| `SecurityModule` | Security/Auth | `forRoot({ jwt: { secret: '...' } })` |
| `DatabaseModule` | Database | `forRoot({ type: 'postgres', ... })` |
| `CacheModule` | Caching | `forRoot({ ttl: 60 })` |
| `SessionModule` | Sessions | `forRoot({ secret: '...' })` |
| `HealthModule` | Health checks | `forRoot({})` |
| `MetricsModule` | Metrics | `forRoot({})` |
| `SwaggerModule` | API docs | `forRoot({ title: '...' })` |
| `AiModule` | LLM providers and Tool Calling | `forRoot({ providers: [...] })` |
| `ConversationModule` | Multi-turn memory | `forRoot({ store: ... })` |
| `PromptModule` | Prompt templates | `forRoot({})` |
| `EmbeddingModule` | Embedding generation | `forRoot({ provider: ... })` |
| `VectorStoreModule` | Vector retrieval | `forRoot({ store: ... })` |
| `RagModule` | RAG pipeline | `forRoot({ collection: 'kb' })` |
| `McpModule` | MCP server | `forRoot({ transport: 'sse', path: '/mcp' })` |
| `AiGuardModule` | AI safety checks | `forRoot({ piiDetection: {...} })` |

## Modular Application Structure

```
src/
├── main.ts
├── app.module.ts          # Root module
├── users/
│   ├── user.module.ts
│   ├── user.controller.ts
│   ├── user.service.ts
│   └── dto/
│       └── create-user.dto.ts
├── orders/
│   ├── order.module.ts
│   ├── order.controller.ts
│   └── order.service.ts
└── common/
    ├── common.module.ts
    └── services/
        └── logger.service.ts
```

## Complete Example

```typescript
// common/common.module.ts
@Module({
  providers: [LoggerService, UtilsService],
  exports: [LoggerService, UtilsService],
})
class CommonModule {}

// users/user.module.ts
@Module({
  imports: [CommonModule],
  controllers: [UserController],
  providers: [UserService],
  exports: [UserService],
})
class UserModule {}

// orders/order.module.ts
@Module({
  imports: [CommonModule, UserModule],
  controllers: [OrderController],
  providers: [OrderService],
})
class OrderModule {}

// app.module.ts
ConfigModule.forRoot({ defaultConfig: {} });
LoggerModule.forRoot({ level: LogLevel.INFO });

@Module({
  imports: [ConfigModule, LoggerModule, UserModule, OrderModule],
})
class AppModule {}

// main.ts
const app = new Application({ port: 3100 });
app.registerModule(AppModule);
app.listen();
```

## Getting Module References

```typescript
import { ModuleRegistry } from "@dangao/bun-server";

const app = new Application({ port: 3100 });
app.registerModule(AppModule);

// Get module reference
const moduleRef = ModuleRegistry.getInstance().getModuleRef(AppModule);

// Resolve services from module container
const userService = moduleRef?.container.resolve(UserService);
```

## Best Practices

1. **Split by feature**: One module per business domain
2. **Explicit exports**: Only export services needed by other modules
3. **Avoid circular imports**: Use shared modules to resolve dependencies
4. **Global modules at root**: Keep `ConfigModule`, `LoggerModule` etc. at root level

## Related Resources

- [Event System Configuration](https://github.com/dangaogit/bun-server/blob/main/skills/events/event-module-setup.md)
- [Dependency Injection](dependency-injection.md)
