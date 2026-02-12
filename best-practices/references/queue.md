# Bun Server Queue

## Setup QueueModule

```typescript
import { Module, QueueModule } from "@dangao/bun-server";

QueueModule.forRoot({
  defaultConcurrency: 5,    // Concurrent job processing
  defaultRetries: 3,        // Retry failed jobs
  defaultBackoff: 1000,     // Backoff between retries (ms)
});

@Module({
  imports: [QueueModule],
})
class AppModule {}
```

## Define Queue Handlers

```typescript
import { Injectable, Queue } from "@dangao/bun-server";

@Injectable()
class EmailQueue {
  @Queue("email:send")
  async handleSendEmail(job: { to: string; subject: string; body: string }) {
    console.log(`Sending email to ${job.to}`);
    await this.emailService.send(job.to, job.subject, job.body);
  }

  @Queue("email:bulk", { concurrency: 2, retries: 5 })
  async handleBulkEmail(job: { recipients: string[]; template: string }) {
    for (const recipient of job.recipients) {
      await this.emailService.sendTemplate(recipient, job.template);
    }
  }
}
```

## Add Jobs to Queue

```typescript
import {
  Injectable,
  Inject,
  QueueService,
  QUEUE_SERVICE_TOKEN,
} from "@dangao/bun-server";

@Injectable()
class UserService {
  constructor(
    @Inject(QUEUE_SERVICE_TOKEN) private readonly queue: QueueService
  ) {}

  async createUser(data: CreateUserDto) {
    const user = await this.db.createUser(data);

    // Add job to queue
    await this.queue.add("email:send", {
      to: user.email,
      subject: "Welcome!",
      body: "Thanks for signing up.",
    });

    return user;
  }

  async sendBulkEmail(recipients: string[], template: string) {
    await this.queue.add("email:bulk", { recipients, template });
  }
}
```

## Job Options

```typescript
await this.queue.add("email:send", payload, {
  delay: 5000,           // Delay 5 seconds before processing
  priority: 1,           // Higher priority (default: 0)
  retries: 5,            // Override default retries
  backoff: 2000,         // Override default backoff
  removeOnComplete: true, // Remove job data after completion
  removeOnFail: false,   // Keep failed job data
});
```

## Cron Jobs (Scheduled Tasks)

```typescript
import { Injectable, Cron } from "@dangao/bun-server";

@Injectable()
class ScheduledTasks {
  // Every minute
  @Cron("* * * * *")
  async everyMinute() {
    console.log("Running every minute");
  }

  // Every hour at minute 0
  @Cron("0 * * * *")
  async everyHour() {
    console.log("Running every hour");
  }

  // Every day at midnight
  @Cron("0 0 * * *")
  async daily() {
    await this.cleanupOldData();
  }

  // Every Monday at 9 AM
  @Cron("0 9 * * 1")
  async weeklyReport() {
    await this.generateWeeklyReport();
  }

  // With options
  @Cron("*/5 * * * *", { name: "health-check", runOnInit: true })
  async healthCheck() {
    await this.checkSystemHealth();
  }
}
```

## Cron Expression Reference

```
┌──────────── minute (0-59)
│ ┌────────── hour (0-23)
│ │ ┌──────── day of month (1-31)
│ │ │ ┌────── month (1-12)
│ │ │ │ ┌──── day of week (0-6, Sunday=0)
│ │ │ │ │
* * * * *

Examples:
"* * * * *"      - Every minute
"*/5 * * * *"    - Every 5 minutes
"0 * * * *"      - Every hour
"0 0 * * *"      - Daily at midnight
"0 0 * * 0"      - Weekly on Sunday
"0 0 1 * *"      - Monthly on 1st
"0 9-17 * * 1-5" - Weekdays 9 AM to 5 PM, every hour
```

## Job Events and Monitoring

```typescript
@Injectable()
class QueueMonitor {
  constructor(
    @Inject(QUEUE_SERVICE_TOKEN) private readonly queue: QueueService
  ) {
    // Listen to queue events
    this.queue.on("completed", (job, result) => {
      console.log(`Job ${job.id} completed:`, result);
    });

    this.queue.on("failed", (job, error) => {
      console.error(`Job ${job.id} failed:`, error);
    });

    this.queue.on("progress", (job, progress) => {
      console.log(`Job ${job.id} progress: ${progress}%`);
    });
  }
}
```

## Job Progress

```typescript
@Queue("import:data")
async handleImport(job: JobData, context: JobContext) {
  const items = job.items;
  const total = items.length;

  for (let i = 0; i < total; i++) {
    await this.processItem(items[i]);
    await context.updateProgress(((i + 1) / total) * 100);
  }
}
```

## Queue Patterns

### Delayed Jobs

```typescript
// Send reminder after 24 hours
await this.queue.add(
  "reminder:send",
  { userId, message },
  { delay: 24 * 60 * 60 * 1000 }
);
```

### Rate Limited Processing

```typescript
@Queue("api:call", { concurrency: 1, backoff: 1000 })
async handleApiCall(job: { endpoint: string }) {
  // Processes one at a time with 1s between calls
  return fetch(job.endpoint);
}
```

### Retry with Exponential Backoff

```typescript
@Queue("webhook:send", {
  retries: 5,
  backoffType: "exponential",
  backoff: 1000, // 1s, 2s, 4s, 8s, 16s
})
async handleWebhook(job: WebhookPayload) {
  await this.sendWebhook(job);
}
```

## Related Resources

- [Queue Example](https://github.com/dangaogit/bun-server/blob/main/examples/02-official-modules/queue-app.ts)
