---
name: bun-server-events
description: Event system guide for Bun Server framework. Use when implementing event-driven architecture, using EventModule, @OnEvent decorator, event emitters, or pub/sub patterns.
---

# Bun Server Event System

## Setup EventModule

```typescript
import { Module, EventModule } from "@dangao/bun-server";

// Configure before module definition
EventModule.forRoot({
  wildcard: true,      // Enable wildcard events (user.*)
  maxListeners: 20,    // Max listeners per event
  onError: (error, event, payload) => {
    console.error(`Event error [${event}]:`, error);
  },
});

@Module({
  imports: [EventModule],
})
class AppModule {}
```

## Event Listeners with @OnEvent

```typescript
import { Injectable, OnEvent } from "@dangao/bun-server";

@Injectable()
class NotificationListener {
  @OnEvent("user.created")
  async handleUserCreated(payload: { userId: string; email: string }) {
    console.log("New user:", payload.userId);
    await this.sendWelcomeEmail(payload.email);
  }

  @OnEvent("user.*") // Wildcard: matches user.created, user.updated, etc.
  handleAllUserEvents(payload: any) {
    console.log("User event received:", payload);
  }

  @OnEvent("order.completed", { async: true })
  async handleOrderCompleted(payload: { orderId: string }) {
    // Async handler - won't block emitter
    await this.processOrder(payload.orderId);
  }
}
```

## Initialize Listeners

**Critical**: Must call `initializeListeners` after `registerModule`:

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

## Emit Events

```typescript
import { Injectable, Inject, EVENT_EMITTER_TOKEN, EventEmitter } from "@dangao/bun-server";

@Injectable()
class UserService {
  constructor(
    @Inject(EVENT_EMITTER_TOKEN) private readonly eventEmitter: EventEmitter
  ) {}

  async createUser(data: CreateUserDto) {
    const user = await this.db.createUser(data);

    // Emit event
    this.eventEmitter.emit("user.created", {
      userId: user.id,
      email: user.email,
    });

    return user;
  }

  async updateUser(id: string, data: UpdateUserDto) {
    const user = await this.db.updateUser(id, data);

    this.eventEmitter.emit("user.updated", {
      userId: user.id,
      changes: data,
    });

    return user;
  }
}
```

## Manual Event Registration

Alternative to `@OnEvent` decorator:

```typescript
@Injectable()
class AuditService {
  constructor(
    @Inject(EVENT_EMITTER_TOKEN) private readonly eventEmitter: EventEmitter
  ) {
    // Register listeners manually
    this.eventEmitter.on("user.created", this.logUserCreation.bind(this));
    this.eventEmitter.on("order.*", this.logOrderEvent.bind(this));
  }

  private logUserCreation(payload: any) {
    console.log("[AUDIT] User created:", payload);
  }

  private logOrderEvent(payload: any) {
    console.log("[AUDIT] Order event:", payload);
  }
}
```

## Event Patterns

### Request-scoped Events

```typescript
@Injectable()
class OrderService {
  constructor(
    @Inject(EVENT_EMITTER_TOKEN) private readonly eventEmitter: EventEmitter
  ) {}

  async processOrder(order: Order) {
    this.eventEmitter.emit("order.processing", { orderId: order.id });

    try {
      await this.chargePayment(order);
      this.eventEmitter.emit("order.payment.success", { orderId: order.id });

      await this.fulfillOrder(order);
      this.eventEmitter.emit("order.completed", { orderId: order.id });
    } catch (error) {
      this.eventEmitter.emit("order.failed", {
        orderId: order.id,
        error: error.message,
      });
      throw error;
    }
  }
}
```

### Typed Events

```typescript
// Define event types
interface UserEvents {
  "user.created": { userId: string; email: string };
  "user.updated": { userId: string; changes: Record<string, any> };
  "user.deleted": { userId: string };
}

// Type-safe listener
@OnEvent("user.created")
handleUserCreated(payload: UserEvents["user.created"]) {
  console.log(payload.userId, payload.email);
}
```

## Common Issues

### "Provider not found for token" Error

Ensure `EventModule.forRoot()` is called:

```typescript
// ❌ Wrong
@Module({ imports: [EventModule] })

// ✅ Correct
EventModule.forRoot({ wildcard: true });
@Module({ imports: [EventModule] })
```

### @OnEvent Not Triggering

Ensure `initializeListeners` is called with all listener classes.

## Related Resources

- [EventModule Setup Issues](../../bun-server/skills/events/event-module-setup.md)
- [Events Documentation](../../bun-server/docs/events.md)
