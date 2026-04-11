# Bun Server Skills

Agent skills for the Bun Server framework (`@dangao/bun-server` v3.x) — a high-performance, decorator-driven DI web framework running on Bun Runtime and Node.js 22+.

## Skill Entry Point

| Skill | Description |
|-------|-------------|
| [best-practices](./best-practices/SKILL.md) | **Start here** - Workflow guide with all references for Bun Server development |

The `best-practices` skill is the single entry point. It organizes all framework knowledge as a step-by-step workflow and references detailed guides in the `references/` directory.

## Reference Documents

All detailed guides are located in `best-practices/references/`:

### Core Framework

| Reference | Description |
|-----------|-------------|
| [quickstart](./best-practices/references/quickstart.md) | Quick start guide for project setup and basic configuration |
| [dependency-injection](./best-practices/references/dependency-injection.md) | DI container, @Injectable, @Inject, scopes, Symbol+Interface pattern |
| [controller-routing](./best-practices/references/controller-routing.md) | Controllers, routing, HTTP methods, parameter binding |
| [module-system](./best-practices/references/module-system.md) | Module organization, imports/exports, forRoot pattern |
| [middleware](./best-practices/references/middleware.md) | Middleware pipeline, interceptors, built-in middleware |
| [guards](./best-practices/references/guards.md) | CanActivate, UseGuards, AuthGuard, RolesGuard, Reflector, ExecutionContext |
| [validation](./best-practices/references/validation.md) | Data validation, DTOs, validation decorators |
| [error-handling](./best-practices/references/error-handling.md) | HttpException, exception filters, error responses |
| [lifecycle](./best-practices/references/lifecycle.md) | OnModuleInit, OnApplicationBootstrap, OnApplicationShutdown, shutdown hooks |
| [platform](./best-practices/references/platform.md) | Platform Adapter (v3.0+), Node.js 22+ support, IWebSocket, IServerHandle |

### Official Modules

| Reference | Description |
|-----------|-------------|
| [security](./best-practices/references/security.md) | Authentication, authorization, JWT, OAuth2, guards, roles |
| [database](./best-practices/references/database.md) | Database connections, ORM, entities, repositories, transactions |
| [cache](./best-practices/references/cache.md) | Caching with @Cacheable, cache eviction, Redis cache |
| [queue](./best-practices/references/queue.md) | Job queues, background tasks, @Cron scheduled tasks |
| [session](./best-practices/references/session.md) | Session management, session stores, user state |
| [events](./best-practices/references/events.md) | Event-driven architecture, EventModule, @OnEvent decorator |
| [websocket](./best-practices/references/websocket.md) | WebSocket gateways, real-time communication (uses IWebSocket<T> in v3) |
| [swagger](./best-practices/references/swagger.md) | API documentation, OpenAPI, Swagger UI |
| [health-metrics](./best-practices/references/health-metrics.md) | Health checks, Prometheus metrics, monitoring |
| [logger](./best-practices/references/logger.md) | Logging, log levels, structured logging |
| [dashboard](./best-practices/references/dashboard.md) | Embedded monitoring dashboard UI with routes, health and system info |
| [debug](./best-practices/references/debug.md) | Request recording, replay and debugging UI |
| [async-module](./best-practices/references/async-module.md) | forRootAsync pattern for ConfigModule, DatabaseModule, CacheModule |
| [cluster](./best-practices/references/cluster.md) | ClusterManager, multi-process scaling, worker crash recovery |
| [testing](./best-practices/references/testing.md) | TestingModule, provider override, integration testing patterns |
| [client](./best-practices/references/client.md) | Type-safe API client generation from route manifests |

### AI Modules (v2.0+)

| Reference | Description |
|-----------|-------------|
| [ai](./best-practices/references/ai.md) | AI module stack: LLM access, RAG, MCP, conversation memory, safety |

### Microservices

| Reference | Description |
|-----------|-------------|
| [microservice](./best-practices/references/microservice.md) | Service discovery, config center, load balancing, circuit breaker, tracing |

### Troubleshooting

| Reference | Description |
|-----------|-------------|
| [troubleshooting](./best-practices/references/troubleshooting.md) | Common issues, debugging techniques, error resolution |

## Usage

These skills are designed to work with AI coding assistants (like Cursor) to provide context-aware help when developing with Bun Server.

### When Skills Are Triggered

The `best-practices` skill is triggered for any Bun Server related task. It then guides the AI to load the appropriate reference documents based on the specific task:

- **Project setup**: quickstart reference
- **DI / services**: dependency-injection reference
- **Routing / controllers**: controller-routing reference
- **Module organization**: module-system reference
- **Middleware / interceptors**: middleware reference
- **Guards / access control**: guards reference
- **Validation / DTOs**: validation reference
- **Error handling**: error-handling reference
- **Lifecycle hooks**: lifecycle reference
- **Platform adapter / Node.js support**: platform reference
- **Authentication / JWT / OAuth2**: security reference
- **Database / ORM**: database reference
- **Caching / Redis**: cache reference
- **Job queues / Cron**: queue reference
- **Sessions**: session reference
- **Events / pub-sub**: events reference
- **WebSocket / real-time**: websocket reference
- **API docs / Swagger**: swagger reference
- **Health checks / metrics**: health-metrics reference
- **Logging**: logger reference
- **Dashboard / monitor UI**: dashboard reference
- **Request replay / debug**: debug reference
- **Async config / forRootAsync**: async-module reference
- **Clustering / multi-process**: cluster reference
- **Testing / provider override**: testing reference
- **Typed API client**: client reference
- **AI modules / RAG / MCP**: ai reference
- **Microservices**: microservice reference
- **Debugging / errors**: troubleshooting reference

## Directory Structure

```
bun-server-skills/
├── README.md
├── LICENSE
└── best-practices/
    ├── SKILL.md                       # Main workflow document
    └── references/
        ├── quickstart.md
        ├── dependency-injection.md
        ├── controller-routing.md
        ├── module-system.md
        ├── middleware.md
        ├── guards.md                  # NEW in v3 skill update
        ├── validation.md
        ├── error-handling.md
        ├── lifecycle.md               # NEW in v3 skill update
        ├── platform.md                # NEW in v3 skill update
        ├── security.md
        ├── database.md
        ├── cache.md
        ├── queue.md
        ├── session.md
        ├── events.md
        ├── websocket.md               # Updated: IWebSocket<T> (v3)
        ├── swagger.md
        ├── health-metrics.md
        ├── logger.md
        ├── dashboard.md
        ├── debug.md
        ├── async-module.md            # NEW in v3 skill update
        ├── cluster.md                 # NEW in v3 skill update
        ├── testing.md
        ├── client.md
        ├── ai.md
        ├── microservice.md
        └── troubleshooting.md
```

## Coverage

This skills collection covers the complete Bun Server framework including:

- **Core**: Application, Context, DI Container, Router, Controllers, Guards, Lifecycle Hooks
- **Platform (v3.0+)**: Platform Adapter Layer, Bun + Node.js 22+ dual-runtime support, `IWebSocket<T>`, `IServerHandle`
- **Modules**: Config, Logger, Security, Database, Cache, Queue, Session, Events, Health, Metrics, Swagger, Dashboard, Debug
- **Async Config**: `forRootAsync()` for ConfigModule, DatabaseModule, CacheModule
- **Scaling**: ClusterManager, multi-process, SO_REUSEPORT
- **AI Modules (v2.0+)**: AiModule, ConversationModule, PromptModule, EmbeddingModule, VectorStoreModule, RagModule, McpModule, AiGuardModule
- **Microservices**: Config Center, Service Registry, Service Client, Load Balancing, Circuit Breaker, Tracing
- **Features**: Validation, WebSocket, Interceptors, Guards, Exception Filters, Testing Toolkit, Typed Client

## v3 Breaking Changes

If upgrading from v2.x, be aware of these breaking changes covered in the skill:

| Change | Before (v2) | After (v3) |
|---|---|---|
| WebSocket client type | `ServerWebSocket<T>` from `bun` | `IWebSocket<T>` from `@dangao/bun-server` |
| `app.getServer()` return | `Bun.Server \| undefined` | `IServerHandle \| undefined` |

See [platform reference](./best-practices/references/platform.md) and [migration guide](https://github.com/dangaogit/bun-server/blob/main/docs/migration.md).

## Related Documentation

- [Bun Server README](https://github.com/dangaogit/bun-server/blob/main/README.md)
- [API Documentation](https://github.com/dangaogit/bun-server/blob/main/docs/api.md)
- [Best Practices](https://github.com/dangaogit/bun-server/blob/main/docs/best-practices.md)
- [AI Modules](https://github.com/dangaogit/bun-server/blob/main/docs/ai.md)
- [Migration Guide](https://github.com/dangaogit/bun-server/blob/main/docs/migration.md)

## Contributing

To add a new reference:

1. Create a new `.md` file in `best-practices/references/`
2. Add the reference link in `best-practices/SKILL.md` under the appropriate workflow step
3. Update this README with the new reference
