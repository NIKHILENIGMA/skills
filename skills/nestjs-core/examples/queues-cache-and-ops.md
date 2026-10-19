# Queues, Events, Cron, Redis Cache-Aside, Rate Limiting & Ops

## 1. Two-Tier Async Pattern (`EventEmitter2` + `@nestjs/bullmq` + `@Cron`)

### Tier 1: Fast In-Process Side Effects (`@nestjs/event-emitter`)
```typescript
// Emit inside Service after DB write:
this.eventEmitter.emit('order.created', { orderId: order.id, userId: order.userId });

// Listener in src/modules/orders/orders.listener.ts
@Injectable()
export class OrdersListener {
  private readonly logger = new Logger(OrdersListener.name);

  @OnEvent('order.created', { async: true })
  async handleOrderCreated(payload: { orderId: string; userId: string }): Promise<void> {
    this.logger.log(`Order created event for ${payload.orderId}`);
  }
}
```

### Tier 2: Durable Redis Queue with Retries & Idempotency (`@nestjs/bullmq`)
```typescript
// Enqueue in Service (deterministic jobId prevents duplicate processing):
await this.emailQueue.add(
  'send-order-receipt',
  { orderId: order.id, email: order.customerEmail },
  {
    jobId: `order-receipt-${order.id}`,
    attempts: 3,
    backoff: { type: 'exponential', delay: 2000 },
    removeOnComplete: true,
  },
);

// Processor in src/jobs/email.processor.ts
import { Processor, WorkerHost } from '@nestjs/bullmq';
import { Logger } from '@nestjs/common';
import { Job } from 'bullmq';

@Processor('email')
export class EmailProcessor extends WorkerHost {
  private readonly logger = new Logger(EmailProcessor.name);

  async process(job: Job<{ orderId: string; email: string }>): Promise<void> {
    this.logger.log(`Processing job ${job.id} (${job.name}) for order ${job.data.orderId}`);
    // Call external email SDK with async/await
  }
}
```

### Scheduled Cron Jobs (`@nestjs/schedule`)
```typescript
@Injectable()
export class StaleOrdersCron {
  private readonly logger = new Logger(StaleOrdersCron.name);

  @Cron(CronExpression.EVERY_HOUR)
  async expirePendingOrders(): Promise<void> {
    this.logger.log('Expiring stale pending orders...');
    // Run async/await Drizzle batch update
  }
}
```

---

## 2. Redis Cache-Aside & Explicit Invalidation on Mutations

```typescript
@Injectable()
export class ProductsService {
  constructor(
    @Inject(DB_PROVIDER) private readonly db: DrizzleDb,
    @Inject(CACHE_MANAGER) private readonly cache: Cache,
  ) {}

  private cacheKey(id: string): string {
    return `product:${id}`;
  }

  async findOne(id: string): Promise<ProductDto> {
    const cached = await this.cache.get<ProductDto>(this.cacheKey(id));
    if (cached) return cached;

    const [row] = await this.db.select().from(products).where(eq(products.id, id)).limit(1);
    if (!row) {
      throw new NotFoundException({ code: 'RESOURCE_NOT_FOUND', message: `Product ${id} not found` });
    }

    const dto = this.toDto(row);
    await this.cache.set(this.cacheKey(id), dto, 60_000); // 60s TTL
    return dto;
  }

  async update(id: string, input: UpdateProductInput): Promise<ProductDto> {
    const [updated] = await this.db
      .update(products)
      .set(input)
      .where(eq(products.id, id))
      .returning();

    if (!updated) {
      throw new NotFoundException({ code: 'RESOURCE_NOT_FOUND', message: `Product ${id} not found` });
    }

    await this.cache.del(this.cacheKey(id)); // Always invalidate cache on mutation
    return this.toDto(updated);
  }
}
```

---

## 3. Rate Limiting (`@nestjs/throttler`)

Register `ThrottlerModule.forRoot([{ ttl: 60_000, limit: 100 }])` and global `APP_GUARD: ThrottlerGuard`. Override on sensitive/public routes:

```typescript
@Public()
@Throttle({ default: { limit: 10, ttl: 60_000 } }) // Strict 10 req/min on webhooks/auth
@Post('webhooks/clerk')
async handleClerkWebhook(@Req() req: RawBodyRequest<Request>) { ... }
```

---

## 4. Production `main.ts` Hardening & `/health` Readiness Endpoint

```typescript
// src/main.ts
import { Logger } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import helmet from 'helmet';
import { AppModule } from './app.module';
import { ApiExceptionFilter } from './common/http/response-envelope';

async function bootstrap() {
  const logger = new Logger('Bootstrap');
  const app = await NestFactory.create(AppModule, { rawBody: true }); // rawBody required for webhook signatures

  app.use(helmet());
  app.enableCors({
    origin: process.env.CORS_ORIGIN?.split(',') ?? false,
    credentials: true,
  });
  app.useGlobalFilters(new ApiExceptionFilter());
  app.enableShutdownHooks();

  const port = Number(process.env.PORT ?? 3000);
  await app.listen(port);
  logger.log(`API listening on port ${port}`);
}
bootstrap();
```

```typescript
// src/modules/health/health.controller.ts
@Controller('v1/health')
export class HealthController {
  constructor(@Inject(DB_PROVIDER) private readonly db: DrizzleDb) {}

  @Public()
  @Get()
  async check() {
    await this.db.execute(sql`SELECT 1`);
    return ok({ status: 'up' }, 'Service is healthy');
  }
}
```
