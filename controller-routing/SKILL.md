---
name: bun-server-controller
description: Controller and routing guide for Bun Server framework. Use when creating controllers, defining routes, using HTTP method decorators (@GET, @POST, etc.), parameter binding (@Body, @Query, @Param), or working with request/response handling.
---

# Bun Server Controllers and Routing

## Basic Controller

```typescript
import {
  Controller,
  GET,
  POST,
  PUT,
  DELETE,
  PATCH,
  Body,
  Query,
  Param,
  Header,
  Ctx,
} from "@dangao/bun-server";

@Controller("/api/users")
class UserController {
  @GET("/")
  list() {
    return [{ id: 1, name: "Alice" }];
  }

  @GET("/:id")
  getById(@Param("id") id: string) {
    return { id, name: "Alice" };
  }

  @POST("/")
  create(@Body() data: CreateUserDto) {
    return { id: "new", ...data };
  }

  @PUT("/:id")
  update(@Param("id") id: string, @Body() data: UpdateUserDto) {
    return { id, ...data };
  }

  @DELETE("/:id")
  delete(@Param("id") id: string) {
    return { deleted: true };
  }
}
```

## Parameter Decorators

```typescript
@Controller("/api")
class ExampleController {
  @POST("/example")
  example(
    // Request body
    @Body() body: any,                    // Full body
    @Body("name") name: string,           // body.name

    // Query parameters ?page=1&size=10
    @Query() query: any,                  // Full query
    @Query("page") page: string,          // query.page

    // Path parameters /users/:id
    @Param("id") id: string,

    // Headers
    @Header("authorization") auth: string,
    @Header() headers: Headers,           // All headers

    // Context object
    @Ctx() ctx: Context,
  ) {
    // ctx provides full request/response access
    return { success: true };
  }
}
```

## Response Handling

### Return Objects Directly (Auto JSON Serialization)

```typescript
@GET("/users")
list() {
  return [{ id: 1, name: "Alice" }];
}
```

### Using ResponseBuilder

```typescript
import { ResponseBuilder, Ctx, Context } from "@dangao/bun-server";

@GET("/download")
download(@Ctx() ctx: Context) {
  return ResponseBuilder.file(ctx, "/path/to/file.pdf");
}

@GET("/redirect")
redirect() {
  return ResponseBuilder.redirect("/new-location", 302);
}

@GET("/custom")
custom() {
  return new Response(JSON.stringify({ data: "custom" }), {
    status: 201,
    headers: { "X-Custom": "header" },
  });
}
```

## Route Features

### Route Parameters

```typescript
@GET("/users/:userId/posts/:postId")
getPost(
  @Param("userId") userId: string,
  @Param("postId") postId: string
) {
  return { userId, postId };
}
```

### Wildcard Routes

```typescript
@GET("/files/*")
serveFile(@Ctx() ctx: Context) {
  const path = ctx.req.url; // Get full path
  return { path };
}
```

### Route Versioning

```typescript
@Controller("/api/v1/users")
class UserV1Controller {
  @GET("/")
  list() {
    return { version: "v1", users: [] };
  }
}

@Controller("/api/v2/users")
class UserV2Controller {
  @GET("/")
  list() {
    return { version: "v2", users: [], meta: {} };
  }
}
```

## Async Handling

```typescript
@Controller("/api")
class AsyncController {
  constructor(private readonly db: DatabaseService) {}

  @GET("/users/:id")
  async getUser(@Param("id") id: string) {
    const user = await this.db.findUser(id);
    if (!user) {
      throw new HttpException(404, "User not found");
    }
    return user;
  }
}
```

## Register Controllers

```typescript
// Method 1: Direct registration
const app = new Application({ port: 3100 });
app.registerController(UserController);

// Method 2: Via module (recommended)
@Module({
  controllers: [UserController],
  providers: [UserService],
})
class UserModule {}
```

## HTTP Status Codes and Exceptions

```typescript
import { HttpException } from "@dangao/bun-server";

@POST("/users")
create(@Body() data: CreateUserDto) {
  if (!data.name) {
    throw new HttpException(400, "Name is required");
  }
  if (await this.userExists(data.email)) {
    throw new HttpException(409, "User already exists");
  }
  return this.userService.create(data);
}
```

## Related Resources

- [Data Validation](../validation/SKILL.md)
- [Middleware](../middleware/SKILL.md)
