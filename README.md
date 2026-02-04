# Bun Server Skills

Agent skills for the Bun Server framework - a high-performance, decorator-driven DI web framework running on Bun Runtime.

## Available Skills

### Core Framework

| Skill | Description |
|-------|-------------|
| [quickstart](./quickstart/SKILL.md) | Quick start guide for project setup and basic configuration |
| [dependency-injection](./dependency-injection/SKILL.md) | DI container, @Injectable, @Inject, scopes, Symbol+Interface pattern |
| [controller-routing](./controller-routing/SKILL.md) | Controllers, routing, HTTP methods, parameter binding |
| [module-system](./module-system/SKILL.md) | Module organization, imports/exports, forRoot pattern |
| [middleware](./middleware/SKILL.md) | Middleware pipeline, interceptors, built-in middleware |
| [validation](./validation/SKILL.md) | Data validation, DTOs, validation decorators |
| [error-handling](./error-handling/SKILL.md) | HttpException, exception filters, error responses |

### Official Modules

| Skill | Description |
|-------|-------------|
| [security](./security/SKILL.md) | Authentication, authorization, JWT, OAuth2, guards, roles |
| [database](./database/SKILL.md) | Database connections, ORM, entities, repositories, transactions |
| [cache](./cache/SKILL.md) | Caching with @Cacheable, cache eviction, Redis cache |
| [queue](./queue/SKILL.md) | Job queues, background tasks, @Cron scheduled tasks |
| [session](./session/SKILL.md) | Session management, session stores, user state |
| [events](./events/SKILL.md) | Event-driven architecture, EventModule, @OnEvent decorator |
| [websocket](./websocket/SKILL.md) | WebSocket gateways, real-time communication |
| [swagger](./swagger/SKILL.md) | API documentation, OpenAPI, Swagger UI |
| [health-metrics](./health-metrics/SKILL.md) | Health checks, Prometheus metrics, monitoring |
| [logger](./logger/SKILL.md) | Logging, log levels, structured logging |

### Microservices

| Skill | Description |
|-------|-------------|
| [microservice](./microservice/SKILL.md) | Service discovery, config center, load balancing, circuit breaker, tracing |

### Troubleshooting

| Skill | Description |
|-------|-------------|
| [troubleshooting](./troubleshooting/SKILL.md) | Common issues, debugging techniques, error resolution |

## Usage

These skills are designed to work with AI coding assistants (like Cursor) to provide context-aware help when developing with Bun Server.

### When Skills Are Triggered

Skills are automatically suggested based on keywords and context:

- **quickstart**: New project setup, initialization, tsconfig configuration
- **dependency-injection**: @Injectable, @Inject, DI container, service registration
- **controller-routing**: @Controller, @GET, @POST, routing, request handling
- **module-system**: @Module, imports, exports, forRoot
- **middleware**: Middleware, interceptors, CORS, rate limiting
- **validation**: @IsString, @IsEmail, DTOs, validation errors
- **error-handling**: HttpException, exception filters, error handling
- **security**: Authentication, JWT, OAuth2, guards, roles
- **database**: Database, ORM, Entity, Repository, Transaction
- **cache**: @Cacheable, @CacheEvict, caching, Redis
- **queue**: @Queue, @Cron, job queue, scheduled tasks
- **session**: Session, session store, user state
- **events**: @OnEvent, EventModule, event emitter
- **websocket**: WebSocket, real-time, chat
- **swagger**: Swagger, OpenAPI, API documentation
- **health-metrics**: Health check, metrics, Prometheus, monitoring
- **logger**: Logging, log levels, LoggerModule
- **microservice**: Service discovery, config center, circuit breaker
- **troubleshooting**: Errors, debugging, "not working", failures

## Directory Structure

```
bun-server-skills/
├── README.md
├── quickstart/
│   └── SKILL.md
├── dependency-injection/
│   └── SKILL.md
├── controller-routing/
│   └── SKILL.md
├── module-system/
│   └── SKILL.md
├── middleware/
│   └── SKILL.md
├── validation/
│   └── SKILL.md
├── error-handling/
│   └── SKILL.md
├── security/
│   └── SKILL.md
├── database/
│   └── SKILL.md
├── cache/
│   └── SKILL.md
├── queue/
│   └── SKILL.md
├── session/
│   └── SKILL.md
├── events/
│   └── SKILL.md
├── websocket/
│   └── SKILL.md
├── swagger/
│   └── SKILL.md
├── health-metrics/
│   └── SKILL.md
├── logger/
│   └── SKILL.md
├── microservice/
│   └── SKILL.md
└── troubleshooting/
    └── SKILL.md
```

## Coverage

This skills collection covers the complete Bun Server framework including:

- **Core**: Application, Context, DI Container, Router, Controllers
- **Modules**: Config, Logger, Security, Database, Cache, Queue, Session, Events, Health, Metrics, Swagger
- **Microservices**: Config Center, Service Registry, Service Client, Load Balancing, Circuit Breaker, Tracing
- **Features**: Validation, WebSocket, Interceptors, Guards, Exception Filters

## Related Documentation

- [Bun Server README](https://github.com/dangaogit/bun-server/blob/main/README.md)
- [API Documentation](https://github.com/dangaogit/bun-server/blob/main/docs/api.md)
- [Best Practices](https://github.com/dangaogit/bun-server/blob/main/docs/best-practices.md)
- [Problem Solutions](https://github.com/dangaogit/bun-server/blob/main/skills/README.md)

## Contributing

To add a new skill:

1. Create a new directory with a descriptive name
2. Add a `SKILL.md` file following the skill format
3. Include YAML frontmatter with `name` and `description`
4. Update this README with the new skill
