---
name: bun-server-error-handling
description: Error handling guide for Bun Server framework. Use when implementing error handling, using HttpException, creating custom exceptions, exception filters, or handling validation errors.
---

# Bun Server Error Handling

## Built-in Exceptions

```typescript
import {
  HttpException,
  BadRequestException,
  UnauthorizedException,
  ForbiddenException,
  NotFoundException,
  InternalServerErrorException,
} from "@dangao/bun-server";

@Controller("/api/users")
class UserController {
  @GET("/:id")
  async getUser(@Param("id") id: string) {
    const user = await this.userService.findById(id);

    if (!user) {
      throw new NotFoundException("User not found");
    }

    return user;
  }

  @POST("/")
  async createUser(@Body() data: CreateUserDto) {
    if (!data.email) {
      throw new BadRequestException("Email is required");
    }

    if (await this.userService.emailExists(data.email)) {
      throw new HttpException(409, "Email already in use");
    }

    return this.userService.create(data);
  }

  @DELETE("/:id")
  async deleteUser(@Param("id") id: string) {
    const user = await this.userService.findById(id);

    if (!user) {
      throw new NotFoundException("User not found");
    }

    if (!this.canDelete(user)) {
      throw new ForbiddenException("Cannot delete this user");
    }

    await this.userService.delete(id);
  }
}
```

## Custom Exceptions

```typescript
import { HttpException } from "@dangao/bun-server";

class UserNotFoundException extends HttpException {
  constructor(userId: string) {
    super(404, `User with ID ${userId} not found`);
    this.name = "UserNotFoundException";
  }
}

class EmailAlreadyExistsException extends HttpException {
  constructor(email: string) {
    super(409, `Email ${email} is already registered`);
    this.name = "EmailAlreadyExistsException";
  }
}

class InsufficientBalanceException extends HttpException {
  constructor(required: number, available: number) {
    super(400, `Insufficient balance: required ${required}, available ${available}`);
    this.name = "InsufficientBalanceException";
  }
}

// Usage
throw new UserNotFoundException("123");
throw new EmailAlreadyExistsException("user@example.com");
```

## Exception Filters

### Global Exception Filter

```typescript
import {
  ExceptionFilter,
  ExceptionFilterRegistry,
  HttpException,
  Context,
} from "@dangao/bun-server";

class GlobalExceptionFilter implements ExceptionFilter {
  catch(error: Error, ctx: Context): Response {
    console.error("Exception caught:", error);

    if (error instanceof HttpException) {
      return new Response(
        JSON.stringify({
          statusCode: error.statusCode,
          message: error.message,
          timestamp: new Date().toISOString(),
          path: ctx.req.url,
        }),
        {
          status: error.statusCode,
          headers: { "Content-Type": "application/json" },
        }
      );
    }

    // Unknown error
    return new Response(
      JSON.stringify({
        statusCode: 500,
        message: "Internal Server Error",
        timestamp: new Date().toISOString(),
      }),
      {
        status: 500,
        headers: { "Content-Type": "application/json" },
      }
    );
  }
}

// Register globally
ExceptionFilterRegistry.getInstance().registerGlobal(new GlobalExceptionFilter());
```

### Type-specific Exception Filter

```typescript
import { ValidationError } from "@dangao/bun-server";

class ValidationExceptionFilter implements ExceptionFilter {
  catch(error: Error, ctx: Context): Response | null {
    if (!(error instanceof ValidationError)) {
      return null; // Pass to next filter
    }

    return new Response(
      JSON.stringify({
        statusCode: 400,
        message: "Validation failed",
        errors: error.errors.map((e) => ({
          field: e.property,
          message: e.message,
          value: e.value,
        })),
      }),
      {
        status: 400,
        headers: { "Content-Type": "application/json" },
      }
    );
  }
}

ExceptionFilterRegistry.getInstance().registerGlobal(
  new ValidationExceptionFilter()
);
```

## Error Handling Middleware

```typescript
import { Context, NextFunction, HttpException } from "@dangao/bun-server";

function errorHandlerMiddleware(ctx: Context, next: NextFunction) {
  try {
    return next();
  } catch (error) {
    // Log error
    console.error("Error:", error);

    // Handle known exceptions
    if (error instanceof HttpException) {
      return new Response(
        JSON.stringify({
          error: error.message,
          statusCode: error.statusCode,
        }),
        { status: error.statusCode }
      );
    }

    // Handle validation errors
    if (error instanceof ValidationError) {
      return new Response(
        JSON.stringify({
          error: "Validation failed",
          details: error.errors,
        }),
        { status: 400 }
      );
    }

    // Unknown error
    return new Response(
      JSON.stringify({ error: "Internal Server Error" }),
      { status: 500 }
    );
  }
}

app.use(errorHandlerMiddleware);
```

## Async Error Handling

```typescript
// Errors in async handlers are automatically caught
@GET("/users/:id")
async getUser(@Param("id") id: string) {
  // This error will be caught and handled
  const user = await this.userService.findById(id);
  if (!user) {
    throw new NotFoundException("User not found");
  }
  return user;
}

// Manual try-catch for custom handling
@POST("/orders")
async createOrder(@Body() data: CreateOrderDto) {
  try {
    return await this.orderService.create(data);
  } catch (error) {
    if (error.code === "PAYMENT_FAILED") {
      throw new BadRequestException("Payment processing failed");
    }
    throw error; // Re-throw other errors
  }
}
```

## Error Response Patterns

### Standard Error Response

```typescript
interface ErrorResponse {
  statusCode: number;
  message: string;
  error?: string;
  timestamp: string;
  path: string;
}

// Example response
{
  "statusCode": 404,
  "message": "User not found",
  "error": "Not Found",
  "timestamp": "2024-01-15T10:30:00.000Z",
  "path": "/api/users/123"
}
```

### Validation Error Response

```typescript
interface ValidationErrorResponse {
  statusCode: 400;
  message: "Validation failed";
  errors: {
    field: string;
    message: string;
    value?: any;
  }[];
}

// Example response
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    { "field": "email", "message": "Invalid email format", "value": "notanemail" },
    { "field": "age", "message": "Must be a positive number", "value": -5 }
  ]
}
```

## Error Codes

```typescript
// Define error codes
export const ErrorCodes = {
  USER_NOT_FOUND: "USER_001",
  EMAIL_EXISTS: "USER_002",
  INVALID_CREDENTIALS: "AUTH_001",
  TOKEN_EXPIRED: "AUTH_002",
  PAYMENT_FAILED: "PAY_001",
} as const;

class CodedHttpException extends HttpException {
  constructor(
    statusCode: number,
    message: string,
    public readonly code: string
  ) {
    super(statusCode, message);
  }
}

// Usage
throw new CodedHttpException(404, "User not found", ErrorCodes.USER_NOT_FOUND);

// Response
{
  "statusCode": 404,
  "message": "User not found",
  "code": "USER_001"
}
```

## Related Resources

- [Error Handling Documentation](https://github.com/dangaogit/bun-server/blob/main/docs/error-handling.md)
- [Validation](../validation/SKILL.md)
