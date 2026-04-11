# Bun Server Guards

Guards control route access by running before the route handler. They implement `CanActivate` and return `boolean | Promise<boolean>`. Unlike middleware, guards have access to `ExecutionContext` with rich request metadata.

## Basic Guard

```typescript
import { Injectable } from '@dangao/bun-server';
import type { CanActivate, ExecutionContext } from '@dangao/bun-server';

@Injectable()
class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = request.getHeader('authorization');
    return !!token;
  }
}
```

## Applying Guards

```typescript
import { Controller, GET, UseGuards } from '@dangao/bun-server';

// Controller level — all methods protected
@Controller('/api/users')
@UseGuards(AuthGuard)
class UserController {
  @GET('/')
  getUsers() { return { users: [] }; }
}

// Method level — selective protection
@Controller('/api')
class ApiController {
  @GET('/public')
  publicEndpoint() { return { message: 'Public' }; }

  @GET('/private')
  @UseGuards(AuthGuard)
  privateEndpoint() { return { message: 'Private' }; }
}
```

## Built-in Guards

```typescript
import {
  AuthGuard,           // Requires authenticated user
  OptionalAuthGuard,   // Validates token if present, allows request regardless
  RolesGuard,          // Checks user roles (use after AuthGuard)
  Roles,               // Decorator to set required roles
} from '@dangao/bun-server';

@Controller('/api/admin')
@UseGuards(AuthGuard, RolesGuard)
class AdminController {
  @GET('/dashboard')
  @Roles('admin')
  dashboard() { return { message: 'Admin Dashboard' }; }

  @GET('/super')
  @Roles('admin', 'superadmin')  // Either role grants access
  superAdmin() { return { message: 'Super Admin' }; }
}
```

## Guard Execution Order

```
HTTP Request
    ↓
Middleware Pipeline
    ↓
Security Filter (token extraction & authentication)
    ↓
Guards: Global → Controller → Method
    ↓
Interceptors (Pre)
    ↓
Route Handler
```

## Global Guards

Register via `SecurityModule`:

```typescript
SecurityModule.forRoot({
  jwt: { secret: 'your-secret' },
  globalGuards: [AuthGuard],    // Applied to all routes
  excludePaths: ['/public', '/health'],
});
```

## Async Guard

```typescript
@Injectable()
class SubscriptionGuard implements CanActivate {
  constructor(private readonly subscriptionService: SubscriptionService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const userId = request.auth?.user?.id;
    if (!userId) return false;

    const subscription = await this.subscriptionService.getSubscription(userId);
    return subscription?.isActive ?? false;
  }
}
```

## Guard with Metadata (Reflector)

```typescript
import { Injectable, Inject } from '@dangao/bun-server';
import { Reflector, REFLECTOR_TOKEN } from '@dangao/bun-server';
import type { CanActivate, ExecutionContext } from '@dangao/bun-server';

@Injectable()
class FeatureFlagGuard implements CanActivate {
  constructor(
    @Inject(REFLECTOR_TOKEN) private readonly reflector: Reflector
  ) {}

  canActivate(context: ExecutionContext): boolean {
    // Get metadata — method overrides class
    const feature = this.reflector.getAllAndOverride<string>(
      'feature',
      context.getClass(),
      context.getMethodName(),
    );
    if (!feature) return true;
    return this.isFeatureEnabled(feature);
  }

  private isFeatureEnabled(feature: string): boolean {
    return true; // Check feature flags
  }
}
```

## ExecutionContext API

```typescript
// HTTP context
const httpHost = context.switchToHttp();
const request = httpHost.getRequest();   // Context object
const response = httpHost.getResponse(); // ResponseBuilder

// Controller and method info
const controllerClass = context.getClass();     // Controller class
const handler = context.getHandler();           // Method function
const methodName = context.getMethodName();     // Method name string

// Metadata access (method first, then class)
const metadata = context.getMetadata<string[]>('roles');
```

## Reflector API

```typescript
// Method overrides class (most common pattern)
const roles = reflector.getAllAndOverride<string[]>(
  ROLES_METADATA_KEY,
  context.getClass(),
  context.getMethodName(),
);

// Merge class and method metadata arrays
const permissions = reflector.getAllAndMerge<string[]>(
  'permissions',
  context.getClass(),
  context.getMethodName(),
);
```

## Custom RolesGuard Factory

```typescript
import { createRolesGuard } from '@dangao/bun-server';

// Require ALL roles instead of ANY
const AllRolesGuard = createRolesGuard({ matchAll: true });

// Custom role extraction
const CustomRolesGuard = createRolesGuard({
  getRoles: (context) => {
    const request = context.switchToHttp().getRequest();
    return request.auth?.user?.permissions || [];
  },
});
```

## Throwing Exceptions from Guards

Prefer throwing over returning `false` — gives better error messages:

```typescript
import { ForbiddenException, UnauthorizedException } from '@dangao/bun-server';

@Injectable()
class StrictAuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();

    if (!request.auth?.isAuthenticated) {
      throw new UnauthorizedException('Authentication required');
    }
    if (!this.hasPermission(request.auth.user)) {
      throw new ForbiddenException('Insufficient permissions');
    }
    return true;
  }
}
```

## Best Practices

- Apply `AuthGuard` before `RolesGuard` — order matters.
- Throw exceptions instead of returning `false` for descriptive error responses.
- Keep guards focused on one concern (auth, roles, feature flags).
- Use `OptionalAuthGuard` for routes that work with or without a token.
- Register `AuthGuard` globally via `SecurityModule.forRoot({ globalGuards: [AuthGuard] })` instead of repeating on every controller.
