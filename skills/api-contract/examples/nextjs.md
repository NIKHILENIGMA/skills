# Next.js App Router End-to-End Implementation (`route.ts` + Service + Zod)

Place shared helpers in `src/lib/api-response.ts` and keep all business logic and Drizzle queries in a separate service file (`src/modules/<feature>/<feature>.service.ts`).

## 1. Envelope Helpers, Error Mapper & Reusable Schemas (`src/lib/api-response.ts`)

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

export class AppError extends Error {
  constructor(
    public readonly status: number,
    public readonly code: string,
    message: string,
    public readonly details?: unknown,
  ) {
    super(message);
  }
}

// Reusable Parameter Schemas (choose based on ID column type)
export const uuidParamSchema = z.object({ id: z.string().uuid() });
export const cuidParamSchema = z.object({ id: z.string().cuid() });
export const slugParamSchema = z.object({ slug: z.string().trim().min(1).max(120) });
export const numericIdParamSchema = z.object({ id: z.coerce.number().int().positive() });

// Reusable Pagination Query Schema
export const paginationQuerySchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  sortBy: z.string().trim().optional(),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
  search: z.string().trim().optional(),
});

export type PaginationQuery = z.infer<typeof paginationQuerySchema>;

export function jsonOk<T>(
  data: T,
  message = 'Request successful',
  status = 200,
  meta?: Record<string, unknown>,
) {
  return NextResponse.json(
    {
      success: true,
      message,
      data,
      ...(meta ? { meta } : {}),
      timestamp: new Date().toISOString(),
    },
    { status },
  );
}

export function jsonPaginated<T>(
  items: T[],
  total: number,
  page: number,
  limit: number,
  message = 'Records fetched successfully',
) {
  return jsonOk(items, message, 200, {
    page,
    limit,
    total,
    totalPages: Math.ceil(total / Math.max(limit, 1)),
  });
}

export function handleApiError(err: unknown, req: NextRequest) {
  if (err instanceof z.ZodError) {
    return NextResponse.json(
      {
        success: false,
        message: 'Invalid request input',
        error: { code: 'VALIDATION_FAILED', details: err.flatten() },
        timestamp: new Date().toISOString(),
        path: req.nextUrl.pathname,
      },
      { status: 400 },
    );
  }
  if (err instanceof AppError) {
    return NextResponse.json(
      {
        success: false,
        message: err.message,
        error: { code: err.code, ...(err.details ? { details: err.details } : {}) },
        timestamp: new Date().toISOString(),
        path: req.nextUrl.pathname,
      },
      { status: err.status },
    );
  }
  return NextResponse.json(
    {
      success: false,
      message: 'Unexpected server error',
      error: { code: 'INTERNAL_ERROR' },
      timestamp: new Date().toISOString(),
      path: req.nextUrl.pathname,
    },
    { status: 500 },
  );
}
```

## 2. DTOs & Isolated Service (`src/modules/orders/orders.service.ts`)

```typescript
import { z } from 'zod';
import { AppError, PaginationQuery } from '@/lib/api-response';

export const createOrderSchema = z
  .object({
    customerEmail: z.string().trim().toLowerCase().email(),
    productId: z.string().uuid(),
    quantity: z.number().int().min(1).max(100),
  })
  .strict();

export type CreateOrderInput = z.infer<typeof createOrderSchema>;

export interface OrderDto {
  id: string;
  customerEmail: string;
  productId: string;
  quantity: number;
  createdAt: string;
}

export async function listOrders(query: PaginationQuery): Promise<{ items: OrderDto[]; total: number }> {
  const offset = (query.page - 1) * query.limit;
  const [rows, [{ count }]] = await Promise.all([
    db.select().from(orders).limit(query.limit).offset(offset),
    db.select({ count: sql<number>`count(*)::int` }).from(orders),
  ]);
  return { items: rows.map(toOrderDto), total: count };
}

export async function getOrderById(id: string): Promise<OrderDto> {
  const [row] = await db.select().from(orders).where(eq(orders.id, id)).limit(1);
  if (!row) {
    throw new AppError(404, 'RESOURCE_NOT_FOUND', `Order ${id} not found`);
  }
  return toOrderDto(row);
}

export async function createOrder(input: CreateOrderInput): Promise<OrderDto> {
  try {
    const [row] = await db.insert(orders).values(input).returning();
    return toOrderDto(row);
  } catch (err: any) {
    if (err?.code === '23505') {
      throw new AppError(409, 'CONFLICT', 'Duplicate order');
    }
    throw err;
  }
}
```

## 3. Thin Route Handlers (`app/api/v1/orders/route.ts` & `[id]/route.ts`)

```typescript
// app/api/v1/orders/route.ts
import { NextRequest } from 'next/server';
import { handleApiError, jsonOk, jsonPaginated, paginationQuerySchema } from '@/lib/api-response';
import { createOrder, createOrderSchema, listOrders } from '@/modules/orders/orders.service';

export async function GET(req: NextRequest) {
  try {
    const query = paginationQuerySchema.parse(Object.fromEntries(req.nextUrl.searchParams));
    const { items, total } = await listOrders(query);
    return jsonPaginated(items, total, query.page, query.limit, 'Orders fetched successfully');
  } catch (err) {
    return handleApiError(err, req);
  }
}

export async function POST(req: NextRequest) {
  try {
    const rawBody = await req.json();
    const input = createOrderSchema.parse(rawBody);
    const created = await createOrder(input);
    return jsonOk(created, 'Order created successfully', 201);
  } catch (err) {
    return handleApiError(err, req);
  }
}
```

```typescript
// app/api/v1/orders/[id]/route.ts
import { NextRequest } from 'next/server';
import { handleApiError, jsonOk, uuidParamSchema } from '@/lib/api-response';
import { getOrderById } from '@/modules/orders/orders.service';

export async function GET(
  req: NextRequest,
  context: { params: Promise<{ id: string }> },
) {
  try {
    const rawParams = await context.params;
    const { id } = uuidParamSchema.parse(rawParams); // swap to cuidParamSchema / numericIdParamSchema as needed
    const order = await getOrderById(id);
    return jsonOk(order, 'Order fetched successfully');
  } catch (err) {
    return handleApiError(err, req);
  }
}
```
