# Bun Server Middleware and Interceptors

## Middleware Basics

### Using Built-in Middleware

```typescript
import {
  Application,
  createCorsMiddleware,
  createLoggerMiddleware,
  createRateLimitMiddleware,
  createStaticMiddleware,
} from "@dangao/bun-server";

const app = new Application({ port: 3100 });

// CORS
app.use(createCorsMiddleware({
  origin: ["http://localhost:3000"],
  methods: ["GET", "POST", "PUT", "DELETE"],
  credentials: true,
}));

// Logging
app.use(createLoggerMiddleware());

// Rate limiting
app.use(createRateLimitMiddleware({
  windowMs: 60000,  // 1 minute
  max: 100,         // max 100 requests
}));

// Static files
app.use(createStaticMiddleware({
  root: "./public",
  prefix: "/static",
}));
```

## Creating Custom Middleware

```typescript
import { Context, NextFunction } from "@dangao/bun-server";

// Function middleware
function authMiddleware(ctx: Context, next: NextFunction) {
  const token = ctx.req.headers.get("authorization");
  if (!token) {
    return new Response("Unauthorized", { status: 401 });
  }
  ctx.set("user", decodeToken(token));
  return next();
}

// Async middleware
async function dbMiddleware(ctx: Context, next: NextFunction) {
  const conn = await getConnection();
  ctx.set("db", conn);
  try {
    return await next();
  } finally {
    await conn.release();
  }
}

// Register
app.use(authMiddleware);
app.use(dbMiddleware);
```

## Middleware Scopes

```typescript
// Global middleware
app.use(globalMiddleware);

// Controller-level middleware
@Controller("/api")
@UseMiddleware(controllerMiddleware)
class ApiController {

  // Method-level middleware
  @GET("/protected")
  @UseMiddleware(methodMiddleware)
  protected() {
    return { data: "secret" };
  }
}
```

## Interceptors

Interceptors can execute logic before and after request handling:

```typescript
import {
  BaseInterceptor,
  InterceptorContext,
  InterceptorNext,
} from "@dangao/bun-server";

class LoggingInterceptor extends BaseInterceptor {
  async intercept(context: InterceptorContext, next: InterceptorNext) {
    const start = Date.now();
    console.log(`[REQ] ${context.method} ${context.path}`);

    const result = await next();

    const duration = Date.now() - start;
    console.log(`[RES] ${context.method} ${context.path} - ${duration}ms`);

    return result;
  }
}

class TransformInterceptor extends BaseInterceptor {
  async intercept(context: InterceptorContext, next: InterceptorNext) {
    const result = await next();
    return {
      data: result,
      timestamp: new Date().toISOString(),
    };
  }
}
```

### Registering Interceptors

```typescript
import { UseInterceptors } from "@dangao/bun-server";

// Controller level
@Controller("/api")
@UseInterceptors(LoggingInterceptor)
class ApiController {

  // Method level
  @GET("/users")
  @UseInterceptors(TransformInterceptor)
  getUsers() {
    return [{ id: 1, name: "Alice" }];
  }
}

// Global registration
import { InterceptorRegistry } from "@dangao/bun-server";

InterceptorRegistry.getInstance().registerGlobal(new LoggingInterceptor());
```

## Built-in Interceptors

```typescript
import {
  CacheInterceptor,
  LogInterceptor,
  PermissionInterceptor,
} from "@dangao/bun-server";

@Controller("/api")
@UseInterceptors(LogInterceptor)
class ApiController {

  @GET("/data")
  @UseInterceptors(CacheInterceptor) // Cache response
  getData() {
    return expensiveOperation();
  }
}
```

## Execution Order

```
Request
  ↓
Global Middleware
  ↓
Controller Middleware
  ↓
Method Middleware
  ↓
Global Interceptors (Pre)
  ↓
Controller Interceptors (Pre)
  ↓
Method Interceptors (Pre)
  ↓
Route Handler
  ↓
Method Interceptors (Post)
  ↓
Controller Interceptors (Post)
  ↓
Global Interceptors (Post)
  ↓
Response
```

## Error Handling Middleware

```typescript
function errorHandlerMiddleware(ctx: Context, next: NextFunction) {
  try {
    return next();
  } catch (error) {
    if (error instanceof HttpException) {
      return new Response(JSON.stringify({
        error: error.message,
        statusCode: error.statusCode,
      }), { status: error.statusCode });
    }
    console.error("Unhandled error:", error);
    return new Response("Internal Server Error", { status: 500 });
  }
}
```

## Common Patterns

### Request ID Tracking

```typescript
function requestIdMiddleware(ctx: Context, next: NextFunction) {
  const requestId = ctx.req.headers.get("x-request-id") || crypto.randomUUID();
  ctx.set("requestId", requestId);
  const response = next();
  if (response instanceof Response) {
    response.headers.set("x-request-id", requestId);
  }
  return response;
}
```

### Timing Middleware

```typescript
async function timingMiddleware(ctx: Context, next: NextFunction) {
  const start = performance.now();
  const response = await next();
  const duration = performance.now() - start;
  if (response instanceof Response) {
    response.headers.set("x-response-time", `${duration.toFixed(2)}ms`);
  }
  return response;
}
```

## Related Resources

- [Guards and Security](security.md)
- [Request Lifecycle](https://github.com/dangaogit/bun-server/blob/main/docs/request-lifecycle.md)
