# Next.js 16+ Data Fetching, Typed `apiClient` (Mode B) & URL State (`nuqs`)

## 1. Typed `apiClient` Unwrapping `ApiResponse<T>` (`src/lib/api-client.ts` — Mode B)

Automatically attaches the Clerk / Supabase Bearer token and unwraps the standard `{ success, message, data, meta, timestamp }` envelope from `api-contract`:

```typescript
import { env } from '@/lib/env';

export interface PaginationMeta {
  page: number;
  limit: number;
  total: number;
  totalPages: number;
}

export interface ApiSuccess<T> {
  success: true;
  message: string;
  data: T;
  meta?: PaginationMeta;
  timestamp: string;
}

export class ApiClientError extends Error {
  constructor(
    public readonly status: number,
    public readonly code: string,
    message: string,
    public readonly details?: unknown,
  ) {
    super(message);
  }
}

async function getAuthToken(): Promise<string | null> {
  // Server-side vs Client-side token extraction (e.g. Clerk auth().getToken() or Supabase session)
  if (typeof window === 'undefined') {
    const { auth } = await import('@clerk/nextjs/server');
    const session = await auth();
    return await session.getToken();
  }
  return null;
}

export async function apiRequest<T>(
  path: string,
  init?: RequestInit,
): Promise<ApiSuccess<T>> {
  const token = await getAuthToken();
  const res = await fetch(`${env.NEXT_PUBLIC_API_URL}${path}`, {
    ...init,
    headers: {
      'Content-Type': 'application/json',
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...init?.headers,
    },
  });

  if (res.status === 204) {
    return { success: true, message: 'Deleted', data: undefined as T, timestamp: new Date().toISOString() };
  }

  const body = await res.json();
  if (!res.ok || !body.success) {
    throw new ApiClientError(
      res.status,
      body?.error?.code ?? 'INTERNAL_ERROR',
      body?.message ?? 'Request failed',
      body?.error?.details,
    );
  }
  return body as ApiSuccess<T>;
}
```

---

## 2. Next.js 16+ Server Component Page (`await params` / `await searchParams`)

```typescript
// src/app/(dashboard)/orders/[id]/page.tsx
import { Suspense } from 'react';
import { notFound } from 'next/navigation';
import { z } from 'zod';
import { Skeleton } from '@/components/ui/skeleton';
import { OrderDetailsCard } from '@/features/orders/components/order-details-card';
import { getOrderById } from '@/features/orders/queries/get-order';

const paramsSchema = z.object({ id: z.string().uuid() });
const searchParamsSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  search: z.string().trim().optional(),
});

interface PageProps {
  params: Promise<{ id: string }>;
  searchParams: Promise<Record<string, string | string[] | undefined>>;
}

export default async function OrderDetailPage({ params, searchParams }: PageProps) {
  const parsedParams = paramsSchema.safeParse(await params);
  if (!parsedParams.success) notFound();

  const filters = searchParamsSchema.parse(await searchParams);

  return (
    <main className="container mx-auto space-y-6 py-6">
      <Suspense fallback={<Skeleton className="h-48 w-full rounded-xl" />}>
        <OrderContent id={parsedParams.data.id} page={filters.page} />
      </Suspense>
    </main>
  );
}

async function OrderContent({ id, page }: { id: string; page: number }) {
  const order = await getOrderById(id, page);
  if (!order) notFound();
  return <OrderDetailsCard order={order} />;
}
```

---

## 3. Client Interactive Fetching with TanStack Query + `nuqs` URL State

```typescript
// src/features/orders/components/orders-table-client.tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { parseAsInteger, parseAsString, useQueryStates } from 'nuqs';
import { apiRequest } from '@/lib/api-client';
import { Input } from '@/components/ui/input';
import { Skeleton } from '@/components/ui/skeleton';
import type { OrderDto } from '@repo/types';

export function OrdersTableClient() {
  const [filters, setFilters] = useQueryStates({
    page: parseAsInteger.withDefault(1),
    search: parseAsString.withDefault(''),
  });

  const { data, isPending } = useQuery({
    queryKey: ['orders', filters.page, filters.search],
    queryFn: () =>
      apiRequest<OrderDto[]>(
        `/v1/orders?page=${filters.page}&limit=20&search=${encodeURIComponent(filters.search)}`,
      ),
  });

  return (
    <section className="space-y-4">
      <Input
        placeholder="Search customer email..."
        value={filters.search}
        onChange={(e) => setFilters({ search: e.target.value, page: 1 })}
        className="max-w-sm"
      />
      {isPending ? (
        <Skeleton className="h-64 w-full rounded-lg" />
      ) : (
        <div className="rounded-lg border border-border bg-card p-4">
          {data?.data.map((order) => (
            <div key={order.id} className="flex items-center justify-between py-2 text-sm">
              <span className="font-medium text-foreground">{order.customerEmail}</span>
              <span className="text-muted-foreground">Qty: {order.quantity}</span>
            </div>
          ))}
        </div>
      )}
    </section>
  );
}
```
