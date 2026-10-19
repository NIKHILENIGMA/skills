# Forms (`react-hook-form` + `zod` + `shadcn`), Server Actions & Two-Layer Auth

## 1. Fail-Fast Environment Validation (`src/lib/env.ts`)

```typescript
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NEXT_PUBLIC_API_URL: z.string().url().optional(),
  DATABASE_URL: z.string().url().optional(), // Required in Mode A (Solo Next.js)
});

export const env = envSchema.parse({
  NODE_ENV: process.env.NODE_ENV,
  NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
  NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
  DATABASE_URL: process.env.DATABASE_URL,
});
```

---

## 2. Two-Layer Authentication (`middleware.ts` + `requireAuth()`)

### Layer 1: Edge Middleware Gate (`src/middleware.ts`)
```typescript
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server';

const isProtectedRoute = createRouteMatcher(['/dashboard(.*)', '/settings(.*)']);

export default clerkMiddleware(async (auth, req) => {
  if (isProtectedRoute(req)) {
    await auth.protect();
  }
});

export const config = {
  matcher: ['/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)', '/(api|trpc)(.*)'],
};
```

### Layer 2: Server Action & Query Auth Guard (`src/lib/auth.ts`)
```typescript
import 'server-only';
import { auth } from '@clerk/nextjs/server';

export interface SessionUser {
  externalId: string;
}

export async function requireAuth(): Promise<SessionUser> {
  const { userId } = await auth();
  if (!userId) {
    throw new Error('UNAUTHORIZED');
  }
  return { externalId: userId };
}
```

---

## 3. Shared Schema & Typed Server Action (`src/features/orders/actions/create-order.action.ts` — Mode A)

```typescript
// src/features/orders/schemas/order.schema.ts
import { z } from 'zod';

export const createOrderSchema = z
  .object({
    customerEmail: z.string().trim().toLowerCase().email('Enter a valid email'),
    productId: z.string().uuid('Select a valid product'),
    quantity: z.coerce.number().int().min(1).max(100),
  })
  .strict();

export type CreateOrderInput = z.infer<typeof createOrderSchema>;
```

```typescript
// src/features/orders/actions/create-order.action.ts
'use server';

import { revalidatePath } from 'next/cache';
import { requireAuth } from '@/lib/auth';
import { db } from '@/db/client';
import { orders } from '@/db/schema';
import { createOrderSchema, type CreateOrderInput } from '../schemas/order.schema';

export interface ActionResult<T = unknown> {
  success: boolean;
  message: string;
  data?: T;
}

export async function createOrderAction(rawInput: CreateOrderInput): Promise<ActionResult<{ id: string }>> {
  try {
    const user = await requireAuth();
    const parsed = createOrderSchema.safeParse(rawInput);
    if (!parsed.success) {
      return { success: false, message: 'Invalid form input' };
    }

    const [created] = await db
      .insert(orders)
      .values({ ...parsed.data, createdBy: user.externalId })
      .returning({ id: orders.id });

    revalidatePath('/dashboard/orders');
    return {
      success: true,
      message: 'Order created successfully',
      data: { id: created.id },
    };
  } catch {
    return { success: false, message: 'Failed to create order' };
  }
}
```

---

## 4. Client Form Component (`react-hook-form` + `zodResolver` + shadcn `<Form>` + `sonner`)

Works identically for **Mode A** (calling `createOrderAction`) or **Mode B** (calling TanStack `useMutation` + `apiRequest`):

```typescript
// src/features/orders/components/create-order-form.tsx
'use client';

import { useTransition } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { toast } from 'sonner';
import { Button } from '@/components/ui/button';
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { createOrderSchema, type CreateOrderInput } from '../schemas/order.schema';
import { createOrderAction } from '../actions/create-order.action';

export function CreateOrderForm({ defaultProductId }: { defaultProductId: string }) {
  const [isPending, startTransition] = useTransition();

  const form = useForm<CreateOrderInput>({
    resolver: zodResolver(createOrderSchema),
    defaultValues: {
      customerEmail: '',
      productId: defaultProductId,
      quantity: 1,
    },
  });

  function onSubmit(values: CreateOrderInput) {
    startTransition(async () => {
      // Mode A: Server Action (For Mode B, replace with useMutation calling apiRequest('/v1/orders'))
      const res = await createOrderAction(values);
      if (!res.success) {
        toast.error(res.message);
        return;
      }
      toast.success(res.message);
      form.reset();
    });
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="customerEmail"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Customer Email</FormLabel>
              <FormControl>
                <Input type="email" placeholder="you@example.com" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <FormField
          control={form.control}
          name="quantity"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Quantity</FormLabel>
              <FormControl>
                <Input type="number" min={1} max={100} {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit" disabled={isPending}>
          {isPending ? 'Creating...' : 'Create Order'}
        </Button>
      </form>
    </Form>
  );
}
```
