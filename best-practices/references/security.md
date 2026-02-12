# Bun Server Security

## Setup SecurityModule

```typescript
import { Module, SecurityModule } from "@dangao/bun-server";

SecurityModule.forRoot({
  jwt: {
    secret: process.env.JWT_SECRET!,
    expiresIn: "1h",
    algorithm: "HS256",
  },
  oauth2: {
    github: {
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
      authorizationUrl: "https://github.com/login/oauth/authorize",
      tokenUrl: "https://github.com/login/oauth/access_token",
      userInfoUrl: "https://api.github.com/user",
      redirectUri: "http://localhost:3100/auth/github/callback",
      scope: ["user:email"],
    },
  },
});

@Module({
  imports: [SecurityModule],
})
class AppModule {}
```

## Guards

### Using Built-in Guards

```typescript
import {
  Controller,
  GET,
  UseGuards,
  Roles,
  AuthGuard,
  RolesGuard,
} from "@dangao/bun-server";

@Controller("/api")
@UseGuards(AuthGuard) // Require authentication
class ApiController {
  @GET("/profile")
  getProfile() {
    return { message: "Authenticated user only" };
  }

  @GET("/admin")
  @UseGuards(RolesGuard)
  @Roles("admin")
  adminOnly() {
    return { message: "Admin only" };
  }

  @GET("/manager")
  @UseGuards(RolesGuard)
  @Roles("admin", "manager") // Either role
  managerArea() {
    return { message: "Admin or manager" };
  }
}
```

### Custom Guard

```typescript
import { CanActivate, ExecutionContext, Injectable } from "@dangao/bun-server";

@Injectable()
class ApiKeyGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean | Promise<boolean> {
    const http = context.switchToHttp();
    const request = http.getRequest();

    const apiKey = request.headers.get("x-api-key");
    return apiKey === process.env.API_KEY;
  }
}

// Usage
@Controller("/api")
@UseGuards(ApiKeyGuard)
class ExternalApiController {}
```

## JWT Authentication

### Generate Token

```typescript
import { Injectable, JWTUtil, Inject, JWT_UTIL_TOKEN } from "@dangao/bun-server";

@Injectable()
class AuthService {
  constructor(@Inject(JWT_UTIL_TOKEN) private readonly jwt: JWTUtil) {}

  async login(email: string, password: string) {
    const user = await this.validateUser(email, password);
    if (!user) {
      throw new UnauthorizedException("Invalid credentials");
    }

    const token = this.jwt.sign({
      sub: user.id,
      email: user.email,
      roles: user.roles,
    });

    return { accessToken: token };
  }
}
```

### Verify Token

```typescript
@Injectable()
class AuthService {
  constructor(@Inject(JWT_UTIL_TOKEN) private readonly jwt: JWTUtil) {}

  verifyToken(token: string) {
    try {
      const payload = this.jwt.verify(token);
      return payload;
    } catch (error) {
      throw new UnauthorizedException("Invalid token");
    }
  }
}
```

## OAuth2 Authentication

```typescript
import {
  Controller,
  GET,
  Query,
  Inject,
  OAuth2Service,
  OAUTH2_SERVICE_TOKEN,
} from "@dangao/bun-server";

@Controller("/auth")
class AuthController {
  constructor(
    @Inject(OAUTH2_SERVICE_TOKEN) private readonly oauth2: OAuth2Service
  ) {}

  @GET("/github")
  githubLogin() {
    const url = this.oauth2.getAuthorizationUrl("github");
    return ResponseBuilder.redirect(url);
  }

  @GET("/github/callback")
  async githubCallback(@Query("code") code: string) {
    const tokens = await this.oauth2.exchangeCode("github", code);
    const userInfo = await this.oauth2.getUserInfo("github", tokens.access_token);

    // Create or update user
    const user = await this.userService.findOrCreate(userInfo);

    // Generate JWT
    const jwt = this.authService.generateToken(user);

    return { accessToken: jwt };
  }
}
```

## Security Context

Access authenticated user info:

```typescript
import {
  SecurityContextHolder,
  Controller,
  GET,
  UseGuards,
  AuthGuard,
} from "@dangao/bun-server";

@Controller("/api")
@UseGuards(AuthGuard)
class UserController {
  @GET("/me")
  getCurrentUser() {
    const context = SecurityContextHolder.getContext();
    const authentication = context.getAuthentication();

    return {
      userId: authentication.getPrincipal().id,
      email: authentication.getPrincipal().email,
      roles: authentication.getAuthorities(),
    };
  }
}
```

## Role-based Access Control

```typescript
@Controller("/api/admin")
@UseGuards(AuthGuard, RolesGuard)
@Roles("admin")
class AdminController {
  @GET("/users")
  listUsers() {
    return this.userService.findAll();
  }

  @DELETE("/users/:id")
  @Roles("super-admin") // Override class-level roles
  deleteUser(@Param("id") id: string) {
    return this.userService.delete(id);
  }
}
```

## Auth Decorator (Legacy)

```typescript
import { Auth, Controller, GET } from "@dangao/bun-server";

@Controller("/api")
class ApiController {
  @GET("/protected")
  @Auth() // Requires authentication
  protected() {
    return { message: "Protected" };
  }

  @GET("/admin")
  @Auth({ roles: ["admin"] })
  adminOnly() {
    return { message: "Admin only" };
  }
}
```

## Security Filter

Apply security globally:

```typescript
import { Application, createSecurityFilter } from "@dangao/bun-server";

const app = new Application({ port: 3100 });

// Add security filter
app.use(createSecurityFilter({
  excludePaths: ["/public", "/health"],
}));
```

## Related Resources

- [Guards Documentation](https://github.com/dangaogit/bun-server/blob/main/docs/guards.md)
- [Auth Example](https://github.com/dangaogit/bun-server/blob/main/examples/02-official-modules/auth-app.ts)
