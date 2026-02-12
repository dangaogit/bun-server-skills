# Bun Server Troubleshooting

## Dependency Injection Issues

### "Parameter type is undefined" Error

**Symptom**:
```
Error: Cannot resolve dependency at index 0 of MyController.
Parameter type is undefined. Use @Inject() decorator to specify the dependency type.
```

**Causes**:
1. Missing `tsconfig.json` configuration
2. Monorepo root missing `tsconfig.json`
3. Debug plugin not reading `tsconfig.json`

**Solution**:

Ensure `tsconfig.json` includes:
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

For monorepo projects, add `tsconfig.json` to root directory.

**Workaround**: Use explicit `@Inject()`:
```typescript
constructor(@Inject(UserService) private userService: UserService) {}
```

### "Provider not found for token" Error

**Symptom**:
```
Error: Provider not found for token: Symbol(@dangao/bun-server:events:emitter)
```

**Cause**: Module requiring `forRoot()` was not configured.

**Solution**:
```typescript
// ❌ Wrong
@Module({
  imports: [EventModule],
})

// ✅ Correct
@Module({
  imports: [EventModule.forRoot({ wildcard: true })],
})
```

## EventModule Issues

### @OnEvent Decorator Not Working

**Cause**: Event listeners not initialized.

**Solution**: Call `initializeListeners` after `registerModule`:

```typescript
import { Application, EventModule, ModuleRegistry } from "@dangao/bun-server";

const app = new Application({ port: 3100 });
app.registerModule(AppModule);

// Initialize event listeners
const rootModuleRef = ModuleRegistry.getInstance().getModuleRef(AppModule);
if (rootModuleRef?.container) {
  EventModule.initializeListeners(
    rootModuleRef.container,
    [NotificationListener, AuditListener], // All @OnEvent classes
  );
}

app.listen();
```

## Application Startup Issues

### Module Configuration Order

**Problem**: Child modules fail to resolve dependencies.

**Solution**: Configure `forRoot()` before module definitions:

```typescript
// ✅ Configure first
EventModule.forRoot({ wildcard: true });
LoggerModule.forRoot({ level: LogLevel.INFO });

// Then define modules
@Module({
  imports: [EventModule],
  providers: [UserService],
})
class UserModule {}
```

### Circular Dependencies

**Symptom**: Stack overflow or undefined services.

**Solution**: Use `forwardRef`:
```typescript
@Injectable()
class ServiceA {
  constructor(
    @Inject(forwardRef(() => ServiceB)) private b: ServiceB
  ) {}
}
```

## Debug Techniques

### Verify Decorator Metadata

```typescript
import "reflect-metadata";

// Check if paramtypes are generated
console.log(
  "paramTypes:",
  Reflect.getMetadata("design:paramtypes", YourController)
);
// Expected: [class ServiceA, class ServiceB, ...]
// Problem: undefined or []
```

### Debug Container Resolution

```typescript
const container = app.getContainer();

// List all registered providers
console.log("Providers:", container.getProviders());

// Try manual resolution
try {
  const service = container.resolve(UserService);
  console.log("Resolved:", service);
} catch (error) {
  console.error("Resolution failed:", error);
}
```

### Debug Module References

```typescript
import { ModuleRegistry } from "@dangao/bun-server";

const moduleRef = ModuleRegistry.getInstance().getModuleRef(AppModule);
console.log("Module container:", moduleRef?.container);
console.log("Module metadata:", moduleRef?.metadata);
```

## Common Mistakes

### Forgetting @Injectable

```typescript
// ❌ Missing decorator
class UserService {
  getUser() { return {}; }
}

// ✅ Correct
@Injectable()
class UserService {
  getUser() { return {}; }
}
```

### Using `import type` with Symbol+Interface Pattern

```typescript
// ❌ Wrong - type-only import removes Symbol at runtime
import type { UserService } from "./user.service";

// ✅ Correct - keeps both interface and Symbol
import { UserService } from "./user.service";
```

### Not Exporting from Module

```typescript
// ❌ Service not accessible to other modules
@Module({
  providers: [SharedService],
})
class SharedModule {}

// ✅ Export to make available
@Module({
  providers: [SharedService],
  exports: [SharedService],
})
class SharedModule {}
```

## Debugging Checklist

1. [ ] `tsconfig.json` has `experimentalDecorators: true`
2. [ ] `tsconfig.json` has `emitDecoratorMetadata: true`
3. [ ] Services have `@Injectable()` decorator
4. [ ] Modules using `forRoot()` are configured before definitions
5. [ ] Services are registered in module `providers`
6. [ ] Required services are in module `exports`
7. [ ] Event listeners call `EventModule.initializeListeners()`

## Related Resources

- [DI Injection Not Working](https://github.com/dangaogit/bun-server/blob/main/skills/di/injection-not-working.md)
- [EventModule Setup](https://github.com/dangaogit/bun-server/blob/main/skills/events/event-module-setup.md)
- [Troubleshooting Docs](https://github.com/dangaogit/bun-server/blob/main/docs/troubleshooting.md)
