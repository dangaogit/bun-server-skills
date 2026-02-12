# Bun Server Swagger

## Setup SwaggerModule

```typescript
import { Module, SwaggerModule } from "@dangao/bun-server";

SwaggerModule.forRoot({
  title: "My API",
  description: "API documentation",
  version: "1.0.0",
  basePath: "/api",
  servers: [
    { url: "http://localhost:3100", description: "Development" },
    { url: "https://api.example.com", description: "Production" },
  ],
  securityDefinitions: {
    bearerAuth: {
      type: "http",
      scheme: "bearer",
      bearerFormat: "JWT",
    },
  },
});

@Module({
  imports: [SwaggerModule],
})
class AppModule {}
```

## API Documentation Decorators

### @ApiTags - Group Endpoints

```typescript
import { Controller, ApiTags } from "@dangao/bun-server";

@Controller("/api/users")
@ApiTags("Users")
class UserController {}
```

### @ApiOperation - Describe Endpoint

```typescript
import { Controller, GET, POST, ApiOperation } from "@dangao/bun-server";

@Controller("/api/users")
@ApiTags("Users")
class UserController {
  @GET("/")
  @ApiOperation({
    summary: "List all users",
    description: "Returns a paginated list of users",
    operationId: "listUsers",
  })
  list() {}

  @POST("/")
  @ApiOperation({
    summary: "Create user",
    description: "Creates a new user account",
  })
  create() {}
}
```

### @ApiParam - Document Path Parameters

```typescript
import { GET, Param, ApiParam } from "@dangao/bun-server";

@GET("/:id")
@ApiOperation({ summary: "Get user by ID" })
@ApiParam({
  name: "id",
  description: "User ID",
  type: "string",
  required: true,
  example: "123e4567-e89b-12d3-a456-426614174000",
})
getById(@Param("id") id: string) {}
```

### @ApiBody - Document Request Body

```typescript
import { POST, Body, ApiBody } from "@dangao/bun-server";

@POST("/")
@ApiOperation({ summary: "Create user" })
@ApiBody({
  description: "User data",
  required: true,
  schema: {
    type: "object",
    properties: {
      name: { type: "string", example: "John Doe" },
      email: { type: "string", format: "email", example: "john@example.com" },
      age: { type: "integer", minimum: 0, example: 25 },
    },
    required: ["name", "email"],
  },
})
create(@Body() data: CreateUserDto) {}
```

### @ApiResponse - Document Responses

```typescript
import { GET, ApiResponse } from "@dangao/bun-server";

@GET("/:id")
@ApiOperation({ summary: "Get user by ID" })
@ApiResponse({
  status: 200,
  description: "User found",
  schema: {
    type: "object",
    properties: {
      id: { type: "string" },
      name: { type: "string" },
      email: { type: "string" },
    },
  },
})
@ApiResponse({
  status: 404,
  description: "User not found",
  schema: {
    type: "object",
    properties: {
      error: { type: "string", example: "User not found" },
    },
  },
})
getById(@Param("id") id: string) {}
```

## Complete Controller Example

```typescript
import {
  Controller,
  GET,
  POST,
  PUT,
  DELETE,
  Body,
  Param,
  Query,
  ApiTags,
  ApiOperation,
  ApiParam,
  ApiBody,
  ApiResponse,
} from "@dangao/bun-server";

@Controller("/api/users")
@ApiTags("Users")
class UserController {
  @GET("/")
  @ApiOperation({ summary: "List users" })
  @ApiResponse({
    status: 200,
    description: "List of users",
    schema: {
      type: "array",
      items: { $ref: "#/components/schemas/User" },
    },
  })
  list(@Query("page") page: number, @Query("limit") limit: number) {
    return this.userService.findAll({ page, limit });
  }

  @GET("/:id")
  @ApiOperation({ summary: "Get user by ID" })
  @ApiParam({ name: "id", type: "string", required: true })
  @ApiResponse({ status: 200, description: "User found" })
  @ApiResponse({ status: 404, description: "User not found" })
  getById(@Param("id") id: string) {
    return this.userService.findById(id);
  }

  @POST("/")
  @ApiOperation({ summary: "Create user" })
  @ApiBody({ description: "User data", required: true })
  @ApiResponse({ status: 201, description: "User created" })
  @ApiResponse({ status: 400, description: "Validation error" })
  create(@Body() data: CreateUserDto) {
    return this.userService.create(data);
  }

  @PUT("/:id")
  @ApiOperation({ summary: "Update user" })
  @ApiParam({ name: "id", type: "string", required: true })
  @ApiBody({ description: "Update data" })
  @ApiResponse({ status: 200, description: "User updated" })
  update(@Param("id") id: string, @Body() data: UpdateUserDto) {
    return this.userService.update(id, data);
  }

  @DELETE("/:id")
  @ApiOperation({ summary: "Delete user" })
  @ApiParam({ name: "id", type: "string", required: true })
  @ApiResponse({ status: 204, description: "User deleted" })
  delete(@Param("id") id: string) {
    return this.userService.delete(id);
  }
}
```

## Swagger UI

Access Swagger UI at `/swagger` (configurable):

```typescript
SwaggerModule.forRoot({
  title: "My API",
  version: "1.0.0",
  path: "/docs",        // Custom path
  jsonPath: "/docs.json", // OpenAPI JSON path
});

// Access UI at: http://localhost:3100/docs
// Access JSON at: http://localhost:3100/docs.json
```

## Authentication in Swagger

```typescript
SwaggerModule.forRoot({
  title: "My API",
  version: "1.0.0",
  securityDefinitions: {
    bearerAuth: {
      type: "http",
      scheme: "bearer",
      bearerFormat: "JWT",
    },
    apiKey: {
      type: "apiKey",
      in: "header",
      name: "X-API-Key",
    },
  },
  security: [{ bearerAuth: [] }], // Global security
});

// Per-endpoint security
@GET("/admin")
@ApiOperation({
  summary: "Admin endpoint",
  security: [{ bearerAuth: [] }],
})
admin() {}
```

## Generate OpenAPI JSON

```typescript
import { SwaggerGenerator } from "@dangao/bun-server";

const spec = SwaggerGenerator.generate({
  title: "My API",
  version: "1.0.0",
  controllers: [UserController, OrderController],
});

// Save to file
Bun.write("openapi.json", JSON.stringify(spec, null, 2));
```

## Related Resources

- [API Documentation](https://github.com/dangaogit/bun-server/blob/main/docs/api.md)
