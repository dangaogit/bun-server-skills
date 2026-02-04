---
name: bun-server-cache
description: Caching guide for Bun Server framework. Use when implementing caching, using CacheModule, @Cacheable decorator, cache eviction, Redis cache, or improving performance.
---

# Bun Server Cache

## Setup CacheModule

```typescript
import { Module, CacheModule } from "@dangao/bun-server";

// Memory cache (default)
CacheModule.forRoot({
  ttl: 60,           // Default TTL in seconds
  max: 1000,         // Max cache entries
});

// Redis cache
CacheModule.forRoot({
  store: "redis",
  redis: {
    host: "localhost",
    port: 6379,
    password: "password",
    db: 0,
  },
  ttl: 300,
});

@Module({
  imports: [CacheModule],
})
class AppModule {}
```

## Declarative Caching

### @Cacheable - Cache Method Results

```typescript
import { Injectable, Cacheable } from "@dangao/bun-server";

@Injectable()
class UserService {
  @Cacheable({ key: "user:{id}", ttl: 300 })
  async getUser(id: string) {
    console.log("Fetching user from database...");
    return this.db.findUser(id);
  }

  @Cacheable({ key: "users:list", ttl: 60 })
  async listUsers() {
    return this.db.findAllUsers();
  }

  // Dynamic key with multiple params
  @Cacheable({ key: "user:{id}:posts:{page}" })
  async getUserPosts(id: string, page: number) {
    return this.db.findUserPosts(id, page);
  }
}
```

### @CacheEvict - Invalidate Cache

```typescript
import { Injectable, Cacheable, CacheEvict } from "@dangao/bun-server";

@Injectable()
class UserService {
  @Cacheable({ key: "user:{id}" })
  async getUser(id: string) {
    return this.db.findUser(id);
  }

  @CacheEvict({ key: "user:{id}" })
  async updateUser(id: string, data: UpdateUserDto) {
    return this.db.updateUser(id, data);
  }

  @CacheEvict({ key: "user:{id}" })
  async deleteUser(id: string) {
    await this.db.deleteUser(id);
  }

  // Evict multiple keys
  @CacheEvict({ key: ["user:{id}", "users:list"] })
  async updateUserAndList(id: string, data: UpdateUserDto) {
    return this.db.updateUser(id, data);
  }

  // Evict all with pattern
  @CacheEvict({ key: "user:*", allEntries: true })
  async clearAllUserCache() {
    // Clears all user:* keys
  }
}
```

### @CachePut - Update Cache

```typescript
import { Injectable, CachePut } from "@dangao/bun-server";

@Injectable()
class UserService {
  // Always executes method and updates cache
  @CachePut({ key: "user:{id}" })
  async updateUser(id: string, data: UpdateUserDto) {
    const user = await this.db.updateUser(id, data);
    return user; // This value is cached
  }
}
```

## Programmatic Caching

```typescript
import {
  Injectable,
  Inject,
  CacheService,
  CACHE_SERVICE_TOKEN,
} from "@dangao/bun-server";

@Injectable()
class ProductService {
  constructor(
    @Inject(CACHE_SERVICE_TOKEN) private readonly cache: CacheService
  ) {}

  async getProduct(id: string) {
    const cacheKey = `product:${id}`;

    // Try cache first
    const cached = await this.cache.get(cacheKey);
    if (cached) {
      return cached;
    }

    // Fetch and cache
    const product = await this.db.findProduct(id);
    await this.cache.set(cacheKey, product, 300); // TTL 300s
    return product;
  }

  async updateProduct(id: string, data: UpdateProductDto) {
    const product = await this.db.updateProduct(id, data);
    await this.cache.delete(`product:${id}`);
    return product;
  }

  async clearProductCache() {
    await this.cache.clear(); // Clear all cache
  }
}
```

## Cache Interceptor

For controller-level caching:

```typescript
import { Controller, GET, UseInterceptors, CacheInterceptor } from "@dangao/bun-server";

@Controller("/api/products")
@UseInterceptors(CacheInterceptor)
class ProductController {
  @GET("/")
  @Cache({ ttl: 60 })
  listProducts() {
    return this.productService.findAll();
  }
}
```

## Enable Cache Proxy

For automatic method-level caching without decorators:

```typescript
import { Injectable, EnableCacheProxy } from "@dangao/bun-server";

@Injectable()
@EnableCacheProxy()
class ExpensiveService {
  // Methods can be cached programmatically
}
```

## Cache Key Patterns

```typescript
// Simple key
@Cacheable({ key: "all-users" })

// Parameter interpolation
@Cacheable({ key: "user:{id}" })
async getUser(id: string) {}

// Multiple parameters
@Cacheable({ key: "search:{query}:page:{page}" })
async search(query: string, page: number) {}

// Object parameter
@Cacheable({ key: "filter:{filter.status}:{filter.category}" })
async filter(filter: { status: string; category: string }) {}

// Custom key generator
@Cacheable({
  keyGenerator: (args) => `custom:${args[0]}:${Date.now()}`,
})
async customKey(id: string) {}
```

## Conditional Caching

```typescript
@Cacheable({
  key: "user:{id}",
  condition: (args) => args[0] !== "admin", // Don't cache admin
  unless: (result) => result === null,       // Don't cache null
})
async getUser(id: string) {}
```

## Redis Cache Store

```typescript
import { RedisCacheStore, CacheModule } from "@dangao/bun-server";

CacheModule.forRoot({
  store: new RedisCacheStore({
    host: "localhost",
    port: 6379,
    password: "password",
    keyPrefix: "myapp:",
    serializer: JSON,
  }),
  ttl: 300,
});
```

## Related Resources

- [Cache Example](https://github.com/dangaogit/bun-server/blob/main/examples/02-official-modules/cache-app.ts)
- [Performance Guide](https://github.com/dangaogit/bun-server/blob/main/docs/performance.md)
