---
name: bun-server-di
description: Dependency injection guide for Bun Server framework. Use when working with DI container, @Injectable, @Inject decorators, service registration, scopes (singleton/transient/scoped), or Symbol+Interface pattern.
---

# Bun Server Dependency Injection

## Basic Usage

### 1. Mark Services with @Injectable

```typescript
import { Injectable } from "@dangao/bun-server";

@Injectable()
class UserService {
  getUser(id: string) {
    return { id, name: "Alice" };
  }
}
```

### 2. Constructor Injection

```typescript
@Injectable()
class OrderService {
  constructor(private readonly userService: UserService) {}

  getOrderWithUser(orderId: string) {
    const user = this.userService.getUser("1");
    return { orderId, user };
  }
}
```

### 3. Register Services

```typescript
// Method 1: Manual registration
const app = new Application({ port: 3100 });
app.getContainer().register(UserService);
app.getContainer().register(OrderService);

// Method 2: Via module (recommended)
@Module({
  providers: [UserService, OrderService],
})
class AppModule {}
```

## Scopes

```typescript
import { Injectable, Scope, SCOPE } from "@dangao/bun-server";

// Singleton (default) - single shared instance globally
@Injectable()
class SingletonService {}

// Transient - new instance on every injection
@Injectable()
@Scope(SCOPE.TRANSIENT)
class TransientService {}

// Request scoped - one instance per request
@Injectable()
@Scope(SCOPE.SCOPED)
class RequestScopedService {}
```

## Symbol + Interface Pattern

Recommended pattern to solve TypeScript type erasure:

```typescript
// 1. Define interface and same-named Symbol
interface UserService {
  find(id: string): Promise<User>;
  create(data: CreateUserDto): Promise<User>;
}
const UserService = Symbol("UserService");

// 2. Implement the interface
@Injectable()
class UserServiceImpl implements UserService {
  async find(id: string) {
    return { id, name: "Alice" };
  }
  async create(data: CreateUserDto) {
    return { id: "new", ...data };
  }
}

// 3. Register in module
@Module({
  providers: [
    {
      provide: UserService,      // Symbol
      useClass: UserServiceImpl, // Implementation
    },
  ],
  exports: [UserService],
})
class UserModule {}

// 4. Inject and use
@Controller("/users")
class UserController {
  constructor(
    @Inject(UserService) private readonly userService: UserService
  ) {}
}
```

**Key**: Use `import { UserService }` not `import type { UserService }`.

## Advanced Registration

```typescript
@Module({
  providers: [
    // Class provider
    UserService,

    // Use class (replaceable implementation)
    { provide: UserService, useClass: UserServiceImpl },

    // Factory function
    {
      provide: "DATABASE_URL",
      useFactory: () => process.env.DATABASE_URL,
    },

    // Factory with dependencies
    {
      provide: CacheService,
      useFactory: (config: ConfigService) => new CacheService(config.get("cache")),
      inject: [ConfigService],
    },

    // Direct value
    { provide: "APP_NAME", useValue: "My App" },
  ],
})
class AppModule {}
```

## Common Issues

### Dependency Injection Fails

If you see `Parameter type is undefined`:

1. Check `tsconfig.json` includes:
   ```json
   {
     "compilerOptions": {
       "experimentalDecorators": true,
       "emitDecoratorMetadata": true
     }
   }
   ```

2. Monorepo projects: ensure root directory also has `tsconfig.json`

3. Fallback: use `@Inject()` decorator explicitly
   ```typescript
   constructor(@Inject(UserService) private userService: UserService) {}
   ```

### Circular Dependencies

Use `forwardRef` to resolve:

```typescript
@Injectable()
class ServiceA {
  constructor(
    @Inject(forwardRef(() => ServiceB)) private b: ServiceB
  ) {}
}
```

## Related Resources

- [Troubleshooting: DI Not Working](../../bun-server/skills/di/injection-not-working.md)
- [Symbol+Interface Pattern Guide](../../bun-server/docs/symbol-interface-pattern.md)
