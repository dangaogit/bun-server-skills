---
name: bun-server-best-practices
description: Index of all available Bun Server skills and best practices guide. Use when looking for framework guidance, discovering available skills, or asking about Bun Server capabilities and recommended patterns.
---

# Bun Server Skills Index & Best Practices

This skill provides an overview of all available skills for the Bun Server framework and guides you to the right resource.

## Available Skills

### Getting Started

| Skill | When to Use |
|-------|-------------|
| [quickstart](../quickstart/SKILL.md) | New project setup, TypeScript configuration, first application |

### Core Framework

| Skill | When to Use |
|-------|-------------|
| [dependency-injection](../dependency-injection/SKILL.md) | @Injectable, @Inject, DI scopes, Symbol+Interface pattern |
| [controller-routing](../controller-routing/SKILL.md) | @Controller, HTTP methods, route parameters, request/response |
| [module-system](../module-system/SKILL.md) | @Module, imports/exports, forRoot pattern, modular architecture |
| [middleware](../middleware/SKILL.md) | Custom middleware, interceptors, CORS, rate limiting |
| [validation](../validation/SKILL.md) | @IsString, @IsEmail, DTOs, custom validators |
| [error-handling](../error-handling/SKILL.md) | HttpException, exception filters, error responses |

### Official Modules

| Skill | When to Use |
|-------|-------------|
| [security](../security/SKILL.md) | JWT, OAuth2, @UseGuards, @Roles, authentication |
| [database](../database/SKILL.md) | DatabaseModule, ORM, @Entity, @Repository, @Transactional |
| [cache](../cache/SKILL.md) | @Cacheable, @CacheEvict, Redis cache |
| [queue](../queue/SKILL.md) | @Queue, @Cron, background jobs, scheduled tasks |
| [session](../session/SKILL.md) | SessionModule, session storage, user state |
| [events](../events/SKILL.md) | EventModule, @OnEvent, event-driven architecture |
| [websocket](../websocket/SKILL.md) | @WebSocketGateway, real-time communication |
| [swagger](../swagger/SKILL.md) | SwaggerModule, OpenAPI, @ApiOperation |
| [health-metrics](../health-metrics/SKILL.md) | HealthModule, MetricsModule, Prometheus |
| [logger](../logger/SKILL.md) | LoggerModule, log levels, structured logging |

### Microservices

| Skill | When to Use |
|-------|-------------|
| [microservice](../microservice/SKILL.md) | Service discovery, config center, circuit breaker, tracing |

### Troubleshooting

| Skill | When to Use |
|-------|-------------|
| [troubleshooting](../troubleshooting/SKILL.md) | Debugging, common errors, "not working" issues |

## Quick Reference by Task

### "I want to create a new project"
→ Start with [quickstart](../quickstart/SKILL.md)

### "I need to add authentication"
→ Use [security](../security/SKILL.md) for JWT/OAuth2 and guards

### "I want to validate user input"
→ Use [validation](../validation/SKILL.md) for DTO validation

### "I need to cache expensive operations"
→ Use [cache](../cache/SKILL.md) for @Cacheable decorator

### "I want to run background jobs"
→ Use [queue](../queue/SKILL.md) for job queues and @Cron

### "I need real-time features"
→ Use [websocket](../websocket/SKILL.md) for WebSocket gateways

### "I want to build microservices"
→ Use [microservice](../microservice/SKILL.md) for service discovery and governance

### "Something is not working"
→ Check [troubleshooting](../troubleshooting/SKILL.md) for common issues

## Best Practices Summary

### Project Structure

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

### Module Configuration Order

```typescript
// 1. Configure global modules FIRST
ConfigModule.forRoot({ defaultConfig: {} });
LoggerModule.forRoot({ level: LogLevel.INFO });
EventModule.forRoot({ wildcard: true });

// 2. THEN define modules
@Module({
  imports: [ConfigModule, LoggerModule, EventModule],
})
class AppModule {}
```

### TypeScript Configuration (Required)

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

### Symbol + Interface Pattern

```typescript
// Define interface and Symbol with same name
interface UserService {
  find(id: string): Promise<User>;
}
const UserService = Symbol("UserService");

// Register in module
@Module({
  providers: [{ provide: UserService, useClass: UserServiceImpl }],
})

// Import as value, not type
import { UserService } from "./user.service"; // ✅
import type { UserService } from "./user.service"; // ❌
```

### Error Handling

```typescript
// Use built-in exceptions
throw new NotFoundException("User not found");
throw new BadRequestException("Invalid input");

// Or custom exceptions
class UserNotFoundException extends HttpException {
  constructor(id: string) {
    super(404, `User ${id} not found`);
  }
}
```

### Testing

```bash
# Run tests in package directory
bun --cwd=packages/bun-server test

# Run specific test
bun --cwd=packages/bun-server test user.test.ts
```

## Related Resources

- [Bun Server GitHub](https://github.com/dangaogit/bun-server)
- [API Documentation](https://github.com/dangaogit/bun-server/blob/main/docs/api.md)
- [Best Practices Guide](https://github.com/dangaogit/bun-server/blob/main/docs/best-practices.md)
- [Examples](https://github.com/dangaogit/bun-server/tree/main/examples)
