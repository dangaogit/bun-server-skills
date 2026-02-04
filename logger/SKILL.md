---
name: bun-server-logger
description: Logging guide for Bun Server framework. Use when implementing logging, using LoggerModule, log levels, structured logging, or configuring log output.
---

# Bun Server Logger

## Setup LoggerModule

```typescript
import { Module, LoggerModule, LogLevel } from "@dangao/bun-server";

LoggerModule.forRoot({
  level: LogLevel.INFO,
  format: "json",           // "json" | "text"
  timestamp: true,
  context: true,
  colorize: true,           // For text format
});

@Module({
  imports: [LoggerModule],
})
class AppModule {}
```

## Log Levels

```typescript
import { LogLevel } from "@dangao/bun-server";

// Available levels (from lowest to highest)
LogLevel.DEBUG   // Detailed debugging information
LogLevel.INFO    // General information
LogLevel.WARN    // Warning messages
LogLevel.ERROR   // Error messages
LogLevel.FATAL   // Fatal errors

// Only logs at or above the configured level are output
LoggerModule.forRoot({
  level: LogLevel.INFO, // DEBUG logs will be ignored
});
```

## Using Logger

### Inject Logger

```typescript
import { Injectable, Inject, Logger, LOGGER_TOKEN } from "@dangao/bun-server";

@Injectable()
class UserService {
  constructor(@Inject(LOGGER_TOKEN) private readonly logger: Logger) {}

  async createUser(data: CreateUserDto) {
    this.logger.info("Creating user", { email: data.email });

    try {
      const user = await this.db.createUser(data);
      this.logger.info("User created", { userId: user.id });
      return user;
    } catch (error) {
      this.logger.error("Failed to create user", {
        email: data.email,
        error: error.message,
      });
      throw error;
    }
  }
}
```

### Logger Methods

```typescript
// Basic logging
logger.debug("Debug message");
logger.info("Info message");
logger.warn("Warning message");
logger.error("Error message");

// With context data
logger.info("User logged in", { userId: "123", ip: "192.168.1.1" });

// With error
logger.error("Operation failed", { error: error.message, stack: error.stack });
```

## Structured Logging

```typescript
// JSON output format
LoggerModule.forRoot({
  level: LogLevel.INFO,
  format: "json",
});

// Output:
// {"level":"info","message":"User created","userId":"123","timestamp":"2024-01-15T10:30:00.000Z"}

// Text output format
LoggerModule.forRoot({
  level: LogLevel.INFO,
  format: "text",
  colorize: true,
});

// Output:
// [2024-01-15T10:30:00.000Z] INFO: User created - userId=123
```

## Logger Extension

```typescript
import { LoggerExtension, Application } from "@dangao/bun-server";

const app = new Application({ port: 3100 });

// Register logger extension
app.use(
  LoggerExtension.create({
    level: LogLevel.INFO,
    format: "json",
  })
);
```

## Request Logging Middleware

```typescript
import { createLoggerMiddleware, createRequestLoggingMiddleware } from "@dangao/bun-server";

const app = new Application({ port: 3100 });

// Simple logger middleware
app.use(createLoggerMiddleware());

// Detailed request logging
app.use(createRequestLoggingMiddleware({
  logBody: false,       // Don't log request body
  logHeaders: false,    // Don't log headers
  excludePaths: ["/health", "/metrics"],
}));
```

## Context-aware Logging

```typescript
@Injectable()
class OrderService {
  constructor(@Inject(LOGGER_TOKEN) private readonly logger: Logger) {}

  async processOrder(orderId: string, ctx: Context) {
    const requestId = ctx.get("requestId");

    // Include request context in all logs
    const log = (level: string, message: string, data?: any) => {
      this.logger[level](message, { requestId, orderId, ...data });
    };

    log("info", "Processing order");
    // ... processing ...
    log("info", "Order processed", { status: "completed" });
  }
}
```

## Log Interceptor

```typescript
import { UseInterceptors, LogInterceptor, Log } from "@dangao/bun-server";

@Controller("/api")
@UseInterceptors(LogInterceptor)
class ApiController {
  @GET("/users")
  @Log({ message: "Fetching users" })
  getUsers() {
    return this.userService.findAll();
  }

  @POST("/orders")
  @Log({ message: "Creating order", logResult: true })
  createOrder(@Body() data: CreateOrderDto) {
    return this.orderService.create(data);
  }
}
```

## Custom Logger Implementation

```typescript
import { Logger, LogEntry } from "@dangao/bun-server";

class CustomLogger implements Logger {
  private externalService: ExternalLogService;

  debug(message: string, data?: Record<string, any>) {
    this.log("debug", message, data);
  }

  info(message: string, data?: Record<string, any>) {
    this.log("info", message, data);
  }

  warn(message: string, data?: Record<string, any>) {
    this.log("warn", message, data);
  }

  error(message: string, data?: Record<string, any>) {
    this.log("error", message, data);
  }

  private log(level: string, message: string, data?: Record<string, any>) {
    const entry: LogEntry = {
      level,
      message,
      timestamp: new Date().toISOString(),
      ...data,
    };

    // Send to external service
    this.externalService.send(entry);

    // Also output locally
    console.log(JSON.stringify(entry));
  }
}

// Register custom logger
@Module({
  providers: [
    { provide: LOGGER_TOKEN, useClass: CustomLogger },
  ],
})
class AppModule {}
```

## Logging Best Practices

```typescript
// DO: Use structured logging
logger.info("Order created", { orderId, userId, total });

// DON'T: Concatenate strings
logger.info(`Order ${orderId} created by ${userId}`);

// DO: Log at appropriate levels
logger.debug("Cache hit", { key });      // Debug info
logger.info("User registered", { userId }); // Business events
logger.warn("Rate limit approaching", { usage: "90%" }); // Warnings
logger.error("Payment failed", { error, orderId }); // Errors

// DO: Include context
logger.error("Database query failed", {
  query: "SELECT * FROM users",
  error: error.message,
  duration: 5000,
});

// DON'T: Log sensitive data
logger.info("User login", { password }); // Never!
```

## Related Resources

- [Middleware](../middleware/SKILL.md)
- [Error Handling](../error-handling/SKILL.md)
