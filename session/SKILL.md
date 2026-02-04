---
name: bun-server-session
description: Session management guide for Bun Server framework. Use when implementing user sessions, using SessionModule, session storage, Redis sessions, or managing user state.
---

# Bun Server Session

## Setup SessionModule

```typescript
import { Module, SessionModule } from "@dangao/bun-server";

// Memory session (development)
SessionModule.forRoot({
  secret: "your-session-secret",
  name: "sessionId",
  maxAge: 24 * 60 * 60 * 1000, // 24 hours
  httpOnly: true,
  secure: false, // true in production
});

// Redis session (production)
SessionModule.forRoot({
  secret: process.env.SESSION_SECRET!,
  store: "redis",
  redis: {
    host: "localhost",
    port: 6379,
    password: "password",
    keyPrefix: "session:",
  },
  name: "sessionId",
  maxAge: 24 * 60 * 60 * 1000,
  httpOnly: true,
  secure: true,
  sameSite: "strict",
});

@Module({
  imports: [SessionModule],
})
class AppModule {}
```

## Using Sessions

### Session Decorator

```typescript
import { Controller, GET, POST, Session } from "@dangao/bun-server";

@Controller("/api")
class UserController {
  @GET("/profile")
  getProfile(@Session() session: SessionData) {
    if (!session.userId) {
      throw new UnauthorizedException("Not logged in");
    }
    return this.userService.findById(session.userId);
  }

  @POST("/login")
  async login(@Body() data: LoginDto, @Session() session: SessionData) {
    const user = await this.authService.validate(data.email, data.password);
    if (!user) {
      throw new UnauthorizedException("Invalid credentials");
    }

    // Store user in session
    session.userId = user.id;
    session.roles = user.roles;

    return { message: "Logged in" };
  }

  @POST("/logout")
  logout(@Session() session: SessionData) {
    // Clear session
    session.userId = undefined;
    session.roles = undefined;

    return { message: "Logged out" };
  }
}
```

### Session Service

```typescript
import {
  Injectable,
  Inject,
  SessionService,
  SESSION_SERVICE_TOKEN,
} from "@dangao/bun-server";

@Injectable()
class AuthService {
  constructor(
    @Inject(SESSION_SERVICE_TOKEN) private readonly sessionService: SessionService
  ) {}

  async createSession(userId: string) {
    return this.sessionService.create({
      userId,
      createdAt: Date.now(),
    });
  }

  async getSession(sessionId: string) {
    return this.sessionService.get(sessionId);
  }

  async destroySession(sessionId: string) {
    await this.sessionService.destroy(sessionId);
  }

  async updateSession(sessionId: string, data: Partial<SessionData>) {
    await this.sessionService.update(sessionId, data);
  }
}
```

## Session Middleware

```typescript
import { createSessionMiddleware } from "@dangao/bun-server";

const app = new Application({ port: 3100 });

// Add session middleware
app.use(createSessionMiddleware({
  secret: "your-secret",
  maxAge: 86400000,
}));
```

## Custom Session Data

```typescript
// Define session data interface
interface SessionData {
  userId?: string;
  email?: string;
  roles?: string[];
  cart?: CartItem[];
  lastActivity?: number;
}

// Use typed session
@GET("/cart")
getCart(@Session() session: SessionData) {
  return session.cart || [];
}

@POST("/cart/add")
addToCart(@Session() session: SessionData, @Body() item: CartItem) {
  session.cart = session.cart || [];
  session.cart.push(item);
  return session.cart;
}
```

## Redis Session Store

```typescript
import { RedisSessionStore, SessionModule } from "@dangao/bun-server";

const redisStore = new RedisSessionStore({
  host: "localhost",
  port: 6379,
  password: "password",
  db: 0,
  keyPrefix: "myapp:session:",
  ttl: 86400, // 24 hours
});

SessionModule.forRoot({
  secret: "your-secret",
  store: redisStore,
});
```

## Session Security

```typescript
SessionModule.forRoot({
  secret: process.env.SESSION_SECRET!, // Use strong secret
  name: "__Host-session",              // Secure cookie name prefix
  maxAge: 3600000,                     // 1 hour
  httpOnly: true,                      // Prevent XSS access
  secure: true,                        // HTTPS only
  sameSite: "strict",                  // CSRF protection
  rolling: true,                       // Reset expiry on activity
});
```

## Session Patterns

### Flash Messages

```typescript
@POST("/login")
async login(@Body() data: LoginDto, @Session() session: SessionData) {
  try {
    await this.authService.login(data);
    session.flash = { type: "success", message: "Welcome back!" };
    return ResponseBuilder.redirect("/dashboard");
  } catch (error) {
    session.flash = { type: "error", message: "Invalid credentials" };
    return ResponseBuilder.redirect("/login");
  }
}

@GET("/dashboard")
dashboard(@Session() session: SessionData) {
  const flash = session.flash;
  session.flash = undefined; // Clear after reading
  return { flash, user: session.user };
}
```

### Session Activity Tracking

```typescript
function activityMiddleware(ctx: Context, next: NextFunction) {
  const session = ctx.get("session");
  if (session) {
    session.lastActivity = Date.now();
  }
  return next();
}

// Check for inactive sessions
@GET("/profile")
getProfile(@Session() session: SessionData) {
  const inactiveLimit = 30 * 60 * 1000; // 30 minutes
  if (Date.now() - session.lastActivity > inactiveLimit) {
    throw new UnauthorizedException("Session expired due to inactivity");
  }
  return this.userService.findById(session.userId);
}
```

## Related Resources

- [Session Example](https://github.com/dangaogit/bun-server/blob/main/examples/02-official-modules/session-app.ts)
- [Security](../security/SKILL.md)
