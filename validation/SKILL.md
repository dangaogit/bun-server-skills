---
name: bun-server-validation
description: Data validation guide for Bun Server framework. Use when validating request data, using validation decorators (@IsString, @IsEmail, etc.), creating DTOs, custom validators, or handling validation errors.
---

# Bun Server Data Validation

## Basic Usage

### Define DTO with Validation Rules

```typescript
import {
  IsString,
  IsEmail,
  IsNumber,
  IsOptional,
  MinLength,
  MaxLength,
  Min,
  Max,
  IsArray,
  IsBoolean,
  IsEnum,
  Matches,
} from "@dangao/bun-server";

class CreateUserDto {
  @IsString()
  @MinLength(2)
  @MaxLength(50)
  name: string;

  @IsEmail()
  email: string;

  @IsNumber()
  @Min(0)
  @Max(150)
  @IsOptional()
  age?: number;

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
  password: string;
}
```

### Use in Controller

```typescript
import { Controller, POST, Body, Validate } from "@dangao/bun-server";

@Controller("/api/users")
class UserController {
  @POST("/")
  @Validate(CreateUserDto)
  createUser(@Body() data: CreateUserDto) {
    return this.userService.create(data);
  }
}
```

## Common Validation Decorators

### Type Validation

```typescript
@IsString()      // Must be string
@IsNumber()      // Must be number
@IsBoolean()     // Must be boolean
@IsArray()       // Must be array
@IsObject()      // Must be object
@IsDate()        // Must be date
```

### String Validation

```typescript
@IsEmail()                       // Email format
@IsUrl()                         // URL format
@IsUUID()                        // UUID format
@MinLength(5)                    // Minimum length
@MaxLength(100)                  // Maximum length
@Matches(/^[a-z]+$/)             // Regex match
@IsNotEmpty()                    // Non-empty string
@IsAlpha()                       // Letters only
@IsAlphanumeric()                // Letters and numbers
```

### Number Validation

```typescript
@Min(0)          // Minimum value
@Max(100)        // Maximum value
@IsPositive()    // Positive number
@IsNegative()    // Negative number
@IsInt()         // Integer
```

### Array Validation

```typescript
@IsArray()
@ArrayMinSize(1)
@ArrayMaxSize(10)
@ArrayUnique()
items: string[];
```

### Enum Validation

```typescript
enum UserRole {
  ADMIN = "admin",
  USER = "user",
}

class UpdateRoleDto {
  @IsEnum(UserRole)
  role: UserRole;
}
```

### Optional and Conditional

```typescript
@IsOptional()           // Optional field
@IsNotEmpty()           // Cannot be empty
@ValidateIf(o => o.type === "email")  // Conditional validation
```

## Nested Object Validation

```typescript
import { ValidateNested, Type } from "@dangao/bun-server";

class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;
}

class CreateUserDto {
  @IsString()
  name: string;

  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;
}
```

## Array Element Validation

```typescript
class CreateOrderDto {
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => OrderItemDto)
  items: OrderItemDto[];
}
```

## Custom Validators

```typescript
import { registerDecorator, ValidationArguments } from "@dangao/bun-server";

function IsStrongPassword() {
  return function (object: any, propertyName: string) {
    registerDecorator({
      name: "isStrongPassword",
      target: object.constructor,
      propertyName: propertyName,
      validator: {
        validate(value: any) {
          if (typeof value !== "string") return false;
          return (
            value.length >= 8 &&
            /[a-z]/.test(value) &&
            /[A-Z]/.test(value) &&
            /\d/.test(value)
          );
        },
        defaultMessage() {
          return "Password must be at least 8 characters with uppercase, lowercase and number";
        },
      },
    });
  };
}

// Usage
class RegisterDto {
  @IsStrongPassword()
  password: string;
}
```

## Custom Error Messages

```typescript
class CreateUserDto {
  @IsString({ message: "Name must be a string" })
  @MinLength(2, { message: "Name must be at least 2 characters" })
  name: string;

  @IsEmail({}, { message: "Please enter a valid email address" })
  email: string;
}
```

## Validation Error Handling

Validation failures throw `ValidationError`:

```typescript
import { ValidationError, HttpException } from "@dangao/bun-server";

// Global error handler
function errorHandler(ctx: Context, next: NextFunction) {
  try {
    return next();
  } catch (error) {
    if (error instanceof ValidationError) {
      return new Response(JSON.stringify({
        statusCode: 400,
        message: "Validation failed",
        errors: error.errors,
      }), { status: 400 });
    }
    throw error;
  }
}
```

## Partial Validation (Update Scenarios)

```typescript
import { PartialType } from "@dangao/bun-server";

// Inherit from CreateUserDto, all fields become optional
class UpdateUserDto extends PartialType(CreateUserDto) {}

// Or define manually
class UpdateUserDto {
  @IsOptional()
  @IsString()
  @MinLength(2)
  name?: string;

  @IsOptional()
  @IsEmail()
  email?: string;
}
```

## Query Parameter Validation

```typescript
class PaginationDto {
  @IsOptional()
  @IsNumber()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @IsNumber()
  @Min(1)
  @Max(100)
  limit?: number = 10;
}

@Controller("/api/users")
class UserController {
  @GET("/")
  @Validate(PaginationDto, "query")
  list(@Query() query: PaginationDto) {
    return this.userService.paginate(query.page, query.limit);
  }
}
```

## Related Resources

- [Controllers and Routing](../controller-routing/SKILL.md)
- [Validation Documentation](https://github.com/dangaogit/bun-server/blob/main/docs/validation.md)
