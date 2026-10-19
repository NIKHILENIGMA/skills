# NestJS End-to-End Implementation (Zod + Controller + Service + Error Filter)

Place shared helpers in `src/common/http/` if they do not already exist.
> **Validation rule:** Default to **Zod** for body, query, and route parameters. Only use `class-validator` if the user or existing codebase explicitly requires it.

## 1. Response Envelope & Exception Filter (`src/common/http/response-envelope.ts`)

```typescript
import { ArgumentsHost, Catch, ExceptionFilter, HttpException, HttpStatus } from '@nestjs/common';

export interface PaginationMeta {
  page: number;
  limit: number;
  total: number;
  totalPages: number;
}

export interface ApiResponse<T> {
  success: true;
  message: string;
  data: T;
  meta?: PaginationMeta;
  timestamp: string;
}

export function ok<T>(
  data: T,
  message = 'Request successful',
  meta?: PaginationMeta,
): ApiResponse<T> {
  return {
    success: true,
    message,
    data,
    ...(meta ? { meta } : {}),
    timestamp: new Date().toISOString(),
  };
}

export function paginated<T>(
  items: T[],
  total: number,
  page: number,
  limit: number,
  message = 'Records fetched successfully',
): ApiResponse<T[]> {
  return ok(items, message, {
    page,
    limit,
    total,
    totalPages: Math.ceil(total / Math.max(limit, 1)),
  });
}

@Catch()
export class ApiExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const res = ctx.getResponse();
    const req = ctx.getRequest();

    const status =
      exception instanceof HttpException ? exception.getStatus() : HttpStatus.INTERNAL_SERVER_ERROR;
    const responseBody =
      exception instanceof HttpException ? (exception.getResponse() as Record<string, unknown>) : null;
    const message = (responseBody?.message as string) ?? 'Unexpected server error';

    res.status(status).json({
      success: false,
      message,
      error: {
        code: (responseBody?.code as string) ?? HttpStatus[status] ?? 'INTERNAL_ERROR',
        ...(responseBody?.details ? { details: responseBody.details } : {}),
      },
      timestamp: new Date().toISOString(),
      path: req.url,
    });
  }
}
```

## 2. Zod Validation Pipe & Reusable Param/Query Schemas (`src/common/http/zod-validation.pipe.ts`)

```typescript
import { BadRequestException, Injectable, PipeTransform } from '@nestjs/common';
import { z, ZodSchema } from 'zod';

@Injectable()
export class ZodValidationPipe<T> implements PipeTransform<unknown, T> {
  constructor(private readonly schema: ZodSchema<T>) {}

  transform(value: unknown): T {
    const parsed = this.schema.safeParse(value);
    if (!parsed.success) {
      throw new BadRequestException({
        code: 'VALIDATION_FAILED',
        message: 'Invalid request input',
        details: parsed.error.flatten(),
      });
    }
    return parsed.data;
  }
}

// Reusable Parameter Schemas (choose based on column type)
export const uuidParamSchema = z.string().uuid();
export const cuidParamSchema = z.string().cuid();
export const slugParamSchema = z.string().trim().min(1).max(120).regex(/^[a-z0-9-]+$/);
export const numericIdParamSchema = z.coerce.number().int().positive();

// Reusable Pagination Query Schema
export const paginationQuerySchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  sortBy: z.string().trim().optional(),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
  search: z.string().trim().optional(),
});

export type PaginationQuery = z.infer<typeof paginationQuerySchema>;
```

## 3. Feature DTOs (`src/orders/dto/order.dto.ts`)

```typescript
import { z } from 'zod';

export const createOrderSchema = z
  .object({
    customerEmail: z.string().trim().toLowerCase().email(),
    productId: z.string().uuid(),
    quantity: z.number().int().min(1).max(100),
    notes: z.string().trim().max(500).optional(),
  })
  .strict();

export const updateOrderSchema = createOrderSchema.partial().strict();

export type CreateOrderInput = z.infer<typeof createOrderSchema>;
export type UpdateOrderInput = z.infer<typeof updateOrderSchema>;

// Explicit Output DTO (never expose internal DB columns)
export interface OrderDto {
  id: string;
  customerEmail: string;
  productId: string;
  quantity: number;
  notes: string | null;
  createdAt: string;
}
```

## 4. Thin Controller (`src/orders/orders.controller.ts`)

```typescript
@Controller('v1/orders')
export class OrdersController {
  constructor(private readonly ordersService: OrdersService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(
    @Body(new ZodValidationPipe(createOrderSchema)) input: CreateOrderInput,
  ): Promise<ApiResponse<OrderDto>> {
    const created = await this.ordersService.create(input);
    return ok(created, 'Order created successfully');
  }

  @Get()
  async findAll(
    @Query(new ZodValidationPipe(paginationQuerySchema)) query: PaginationQuery,
  ): Promise<ApiResponse<OrderDto[]>> {
    const { items, total } = await this.ordersService.findAll(query);
    return paginated(items, total, query.page, query.limit, 'Orders fetched successfully');
  }

  @Get(':id')
  async findOne(
    @Param('id', new ZodValidationPipe(uuidParamSchema)) id: string, // swap to cuidParamSchema / numericIdParamSchema as needed
  ): Promise<ApiResponse<OrderDto>> {
    const order = await this.ordersService.findOne(id);
    return ok(order, 'Order fetched successfully');
  }

  @Patch(':id')
  async update(
    @Param('id', new ZodValidationPipe(uuidParamSchema)) id: string,
    @Body(new ZodValidationPipe(updateOrderSchema)) input: UpdateOrderInput,
  ): Promise<ApiResponse<OrderDto>> {
    const updated = await this.ordersService.update(id, input);
    return ok(updated, 'Order updated successfully');
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(
    @Param('id', new ZodValidationPipe(uuidParamSchema)) id: string,
  ): Promise<void> {
    await this.ordersService.remove(id);
  }
}
```

## 5. Isolated Service with `async/await` & Proper Error Handling (`src/orders/orders.service.ts`)

```typescript
@Injectable()
export class OrdersService {
  constructor(@Inject(DB) private readonly db: DrizzleDb) {}

  async create(input: CreateOrderInput): Promise<OrderDto> {
    try {
      const [row] = await this.db.insert(orders).values(input).returning();
      return this.toDto(row);
    } catch (err: any) {
      if (err?.code === '23505') {
        throw new ConflictException({ code: 'CONFLICT', message: 'Order already exists' });
      }
      throw err;
    }
  }

  async findAll(query: PaginationQuery): Promise<{ items: OrderDto[]; total: number }> {
    const offset = (query.page - 1) * query.limit;
    const whereClause = query.search ? ilike(orders.customerEmail, `%${query.search}%`) : undefined;

    // Only use Promise.all for independent parallel queries
    const [rows, [{ count }]] = await Promise.all([
      this.db.select().from(orders).where(whereClause).limit(query.limit).offset(offset),
      this.db.select({ count: sql<number>`count(*)::int` }).from(orders).where(whereClause),
    ]);

    return { items: rows.map((r) => this.toDto(r)), total: count };
  }

  async findOne(id: string): Promise<OrderDto> {
    const [row] = await this.db.select().from(orders).where(eq(orders.id, id)).limit(1);
    if (!row) {
      throw new NotFoundException({ code: 'RESOURCE_NOT_FOUND', message: `Order ${id} not found` });
    }
    return this.toDto(row);
  }

  private toDto(row: typeof orders.$inferSelect): OrderDto {
    return {
      id: row.id,
      customerEmail: row.customerEmail,
      productId: row.productId,
      quantity: row.quantity,
      notes: row.notes,
      createdAt: row.createdAt.toISOString(),
    };
  }
}
```
