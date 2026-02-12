# Bun Server Database

## Setup DatabaseModule

```typescript
import { Module, DatabaseModule } from "@dangao/bun-server";

// SQLite
DatabaseModule.forRoot({
  type: "sqlite",
  database: "./data.db",
});

// PostgreSQL
DatabaseModule.forRoot({
  type: "postgres",
  host: "localhost",
  port: 5432,
  database: "myapp",
  username: "user",
  password: "password",
  pool: {
    min: 2,
    max: 10,
  },
});

// MySQL
DatabaseModule.forRoot({
  type: "mysql",
  host: "localhost",
  port: 3306,
  database: "myapp",
  username: "user",
  password: "password",
});

@Module({
  imports: [DatabaseModule],
})
class AppModule {}
```

## Using DatabaseService

```typescript
import {
  Injectable,
  Inject,
  DatabaseService,
  DATABASE_SERVICE_TOKEN,
} from "@dangao/bun-server";

@Injectable()
class UserRepository {
  constructor(
    @Inject(DATABASE_SERVICE_TOKEN) private readonly db: DatabaseService
  ) {}

  async findAll() {
    return this.db.query("SELECT * FROM users");
  }

  async findById(id: string) {
    return this.db.queryOne("SELECT * FROM users WHERE id = ?", [id]);
  }

  async create(data: CreateUserDto) {
    const result = await this.db.execute(
      "INSERT INTO users (name, email) VALUES (?, ?)",
      [data.name, data.email]
    );
    return { id: result.lastInsertRowid, ...data };
  }

  async update(id: string, data: UpdateUserDto) {
    await this.db.execute(
      "UPDATE users SET name = ?, email = ? WHERE id = ?",
      [data.name, data.email, id]
    );
  }

  async delete(id: string) {
    await this.db.execute("DELETE FROM users WHERE id = ?", [id]);
  }
}
```

## ORM with Entities

### Define Entity

```typescript
import { Entity, Column, PrimaryKey } from "@dangao/bun-server";

@Entity("users")
class User {
  @PrimaryKey()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  @Column({ nullable: true })
  avatar?: string;

  @Column({ default: () => "CURRENT_TIMESTAMP" })
  createdAt: Date;
}
```

### Create Repository

```typescript
import { Repository, BaseRepository, Injectable } from "@dangao/bun-server";

@Repository(User)
@Injectable()
class UserRepository extends BaseRepository<User> {
  async findByEmail(email: string) {
    return this.findOne({ where: { email } });
  }

  async findActive() {
    return this.find({ where: { active: true } });
  }
}
```

### Use Repository

```typescript
@Injectable()
class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  async getUser(id: number) {
    return this.userRepository.findById(id);
  }

  async createUser(data: CreateUserDto) {
    return this.userRepository.create(data);
  }

  async updateUser(id: number, data: UpdateUserDto) {
    return this.userRepository.update(id, data);
  }

  async deleteUser(id: number) {
    return this.userRepository.delete(id);
  }
}
```

## Transactions

### Declarative Transactions

```typescript
import { Injectable, Transactional, Propagation } from "@dangao/bun-server";

@Injectable()
class OrderService {
  constructor(
    private readonly orderRepository: OrderRepository,
    private readonly inventoryService: InventoryService,
    private readonly paymentService: PaymentService
  ) {}

  @Transactional()
  async createOrder(data: CreateOrderDto) {
    // All operations in single transaction
    const order = await this.orderRepository.create(data);
    await this.inventoryService.reserve(data.items);
    await this.paymentService.charge(data.payment);
    return order;
  }

  @Transactional({ propagation: Propagation.REQUIRES_NEW })
  async processRefund(orderId: string) {
    // New transaction regardless of existing
    await this.orderRepository.markRefunded(orderId);
    await this.paymentService.refund(orderId);
  }
}
```

### Programmatic Transactions

```typescript
import {
  Injectable,
  Inject,
  TransactionManager,
  TRANSACTION_SERVICE_TOKEN,
} from "@dangao/bun-server";

@Injectable()
class OrderService {
  constructor(
    @Inject(TRANSACTION_SERVICE_TOKEN)
    private readonly txManager: TransactionManager
  ) {}

  async createOrder(data: CreateOrderDto) {
    return this.txManager.runInTransaction(async (tx) => {
      const order = await tx.query("INSERT INTO orders ...");
      await tx.query("UPDATE inventory ...");
      return order;
    });
  }
}
```

## Transaction Options

```typescript
import { Transactional, Propagation, IsolationLevel } from "@dangao/bun-server";

@Transactional({
  propagation: Propagation.REQUIRED,        // Default
  isolation: IsolationLevel.READ_COMMITTED, // Default
  timeout: 30000,                           // 30 seconds
  readOnly: false,
})
async someMethod() {}

// Propagation options:
// - REQUIRED: Use existing or create new
// - REQUIRES_NEW: Always create new
// - SUPPORTS: Use existing if available
// - NOT_SUPPORTED: Execute without transaction
// - NEVER: Throw if transaction exists
// - NESTED: Nested transaction (savepoint)
```

## Health Indicator

```typescript
import { HealthModule, DatabaseHealthIndicator } from "@dangao/bun-server";

HealthModule.forRoot({
  indicators: [DatabaseHealthIndicator],
});

// GET /health will include database status
// { "database": { "status": "up", "responseTime": 5 } }
```

## Connection Pool

```typescript
DatabaseModule.forRoot({
  type: "postgres",
  // ...
  pool: {
    min: 2,           // Minimum connections
    max: 10,          // Maximum connections
    idleTimeout: 30000, // Close idle connections after 30s
    acquireTimeout: 10000, // Timeout to acquire connection
  },
});
```

## Related Resources

- [Database Example](https://github.com/dangaogit/bun-server/blob/main/examples/02-official-modules/database-app.ts)
- [ORM Example](https://github.com/dangaogit/bun-server/blob/main/examples/02-official-modules/orm-app.ts)
