---
name: bun-server-quickstart
description: Quick start guide for Bun Server framework. Use when creating new Bun Server projects, setting up applications, configuring TypeScript, or asking about project initialization and basic setup.
---

# Bun Server Quick Start

## Prerequisites

- Bun ≥ `1.3.3`

## Critical Configuration

### tsconfig.json (Required)

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

**Critical**: Both `experimentalDecorators` and `emitDecoratorMetadata` must be enabled, otherwise dependency injection will fail.

## Minimal Application

```typescript
import { Application, Controller, GET, Injectable } from "@dangao/bun-server";

@Injectable()
class HealthService {
  ping() {
    return { status: "ok" };
  }
}

@Controller("/api")
class HealthController {
  constructor(private readonly service: HealthService) {}

  @GET("/health")
  check() {
    return this.service.ping();
  }
}

const app = new Application({ port: 3100 });
app.getContainer().register(HealthService);
app.registerController(HealthController);
app.listen();
```

## Modular Application

```typescript
import {
  Application,
  Module,
  Controller,
  GET,
  Injectable,
  ConfigModule,
} from "@dangao/bun-server";

@Injectable()
class UserService {
  getUsers() {
    return [{ id: 1, name: "Alice" }];
  }
}

@Controller("/users")
class UserController {
  constructor(private readonly userService: UserService) {}

  @GET("/")
  list() {
    return this.userService.getUsers();
  }
}

@Module({
  imports: [ConfigModule.forRoot({ defaultConfig: {} })],
  controllers: [UserController],
  providers: [UserService],
})
class AppModule {}

const app = new Application({ port: 3100 });
app.registerModule(AppModule);
app.listen();
```

## Recommended Project Structure

```
src/
├── main.ts              # Entry point
├── app.module.ts        # Root module
├── users/
│   ├── user.module.ts
│   ├── user.controller.ts
│   └── user.service.ts
└── common/
    └── middleware/
```

## Common Imports

```typescript
// Core
import { Application, Controller, Module, Injectable } from "@dangao/bun-server";

// HTTP decorators
import { GET, POST, PUT, DELETE, PATCH } from "@dangao/bun-server";

// Parameter decorators
import { Body, Query, Param, Header, Ctx } from "@dangao/bun-server";

// DI
import { Inject, Scope, SCOPE } from "@dangao/bun-server";

// Modules
import {
  ConfigModule,
  EventModule,
  LoggerModule,
  SecurityModule,
} from "@dangao/bun-server";
```

## Run Commands

```bash
# Install dependencies
bun install

# Run application
bun run src/main.ts

# Run tests (in package directory)
bun --cwd=packages/bun-server test
```

## Next Steps

- [Dependency Injection](../dependency-injection/SKILL.md)
- [Controllers and Routing](../controller-routing/SKILL.md)
- [Module System](../module-system/SKILL.md)
