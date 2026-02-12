---
name: bun-server-best-practices
description: MUST be used for Bun Server tasks. Covers project setup, DI, controllers, modules, middleware, validation, error handling, and all official modules. Load for any Bun Server, @dangao/bun-server, decorator-driven DI, or framework-related work. ALWAYS follow the workflow steps below.
---

# Bun Server Best Practices Workflow

Use this skill as an instruction set. Follow the workflow in order unless the user explicitly asks for a different order.

## Core Principles

- **Decorator-driven architecture**: Use decorators (`@Injectable`, `@Controller`, `@Module`) as the primary configuration mechanism.
- **Explicit dependency injection**: Keep DI contracts clear with Symbol+Interface pattern when needed.
- **Modular organization**: Split by feature domain, one module per business boundary.
- **Convention over configuration**: Follow established patterns for project structure and module setup.
- **Type safety**: Leverage TypeScript decorators and typed contracts throughout.

## 1) Confirm architecture before coding (required)

- Default stack: Bun Runtime + `@dangao/bun-server` + TypeScript with decorators enabled.
- Verify `tsconfig.json` has `experimentalDecorators: true` and `emitDecoratorMetadata: true`.

### 1.1 Must-read core references (required)

Before implementing any Bun Server task, make sure to read and apply these core references:

- [quickstart](references/quickstart.md) - Project setup, minimal/modular application, common imports
- [dependency-injection](references/dependency-injection.md) - @Injectable, @Inject, scopes, Symbol+Interface pattern
- [module-system](references/module-system.md) - @Module, imports/exports, forRoot pattern, modular architecture

Keep these references in active working context for the entire task.

### 1.2 Plan module boundaries before coding (required)

Create a brief module map before implementation for any non-trivial feature:

- Define each module's responsibility in one sentence.
- Identify which modules need `forRoot()` configuration (must be called before module definitions).
- Define provider/export contracts between modules.
- Follow the recommended project structure:

```
src/
├── main.ts                 # Entry point
├── app.module.ts           # Root module
├── common/                 # Shared utilities
│   ├── middleware/
│   ├── filters/
│   └── guards/
├── users/                  # Feature module
│   ├── user.module.ts
│   ├── user.controller.ts
│   ├── user.service.ts
│   └── dto/
└── orders/                 # Feature module
    ├── order.module.ts
    └── ...
```

## 2) Apply core framework foundations (required)

These are essential foundations. Apply all of them in every Bun Server task using the core references already loaded in section `1.1`.

### Dependency Injection

- Must-read reference from `1.1`: [dependency-injection](references/dependency-injection.md)
- Mark all services with `@Injectable()`.
- Use constructor injection as the primary injection method.
- Use Symbol+Interface pattern for abstract service contracts.
- Choose appropriate scopes: Singleton (default), Transient, or Request-scoped.

### Controllers and Routing

- Must-read reference: [controller-routing](references/controller-routing.md)
- Use `@Controller` with route prefix for grouping endpoints.
- Use HTTP method decorators (`@GET`, `@POST`, `@PUT`, `@DELETE`) for route definitions.
- Use parameter decorators (`@Body`, `@Query`, `@Param`, `@Header`, `@Ctx`) for request data binding.
- Return objects directly for auto JSON serialization; use `ResponseBuilder` for special responses.

### Middleware and Interceptors

- Must-read reference: [middleware](references/middleware.md)
- Use built-in middleware for common concerns (CORS, logging, rate limiting, static files).
- Create custom middleware for cross-cutting concerns.
- Use interceptors for pre/post request processing (logging, transformation, caching).
- Follow the execution order: Global -> Controller -> Method for both middleware and interceptors.

### Validation

- Must-read reference: [validation](references/validation.md)
- Define DTOs with validation decorators (`@IsString`, `@IsEmail`, `@Min`, etc.).
- Use `@Validate(DtoClass)` on controller methods.
- Use `ValidateNested` + `@Type()` for nested objects.
- Use `PartialType()` for update DTOs.

### Error Handling

- Must-read reference: [error-handling](references/error-handling.md)
- Use built-in exceptions (`NotFoundException`, `BadRequestException`, etc.).
- Create custom exceptions extending `HttpException` for domain-specific errors.
- Register global exception filters for consistent error responses.
- Async errors in handlers are automatically caught.

## 3) Load official modules only when requirements call for them

Do not add these by default. Load the matching reference only when the requirement exists.

### Authentication and Authorization

- JWT, OAuth2, guards, roles, access control -> [security](references/security.md)
- Session management, session stores, user state -> [session](references/session.md)

### Data and Storage

- Database connections, ORM, entities, repositories, transactions -> [database](references/database.md)
- Caching with @Cacheable, cache eviction, Redis cache -> [cache](references/cache.md)

### Async Processing

- Job queues, background tasks, @Cron scheduled tasks -> [queue](references/queue.md)
- Event-driven architecture, EventModule, @OnEvent -> [events](references/events.md)

### Communication

- WebSocket gateways, real-time features -> [websocket](references/websocket.md)

### Documentation and Observability

- API documentation, OpenAPI, Swagger UI -> [swagger](references/swagger.md)
- Health checks, Prometheus metrics, monitoring -> [health-metrics](references/health-metrics.md)
- Logging, log levels, structured logging -> [logger](references/logger.md)

## 4) Microservice extensions (only when building distributed systems)

Only load when the project explicitly requires microservice architecture:

- Service discovery, config center, load balancing, circuit breaker, distributed tracing -> [microservice](references/microservice.md)

## 5) Final self-check before finishing

- Core behavior works and matches requirements.
- All must-read references from `1.1` were read and applied.
- `tsconfig.json` has `experimentalDecorators` and `emitDecoratorMetadata` enabled.
- DI contracts are explicit: `@Injectable()` on all services, proper `@Inject()` where needed.
- Module boundaries are clear: providers registered, exports declared for cross-module usage.
- `forRoot()` calls happen before module definitions for all configurable modules.
- Controllers use proper parameter decorators and return typed responses.
- Validation DTOs are defined and applied to controller methods.
- Error handling follows framework patterns (HttpException, exception filters).
- Optional modules are used only when requirements demand them.
- If something is not working, check [troubleshooting](references/troubleshooting.md).
