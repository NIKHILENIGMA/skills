# NestJS Solo vs. Monorepo DB, External Auth Guard, Webhook Sync & Decorators

## 1. Database Schema Barrel (`schema/index.ts`) & Drizzle Config

```typescript
// schema/users.schema.ts
import { pgTable, text, timestamp, uuid } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  externalId: text('external_id').notNull().unique(), // Clerk / Supabase / Auth0 sub
  email: text('email').notNull().unique(),
  role: text('role').default('user').notNull(),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
});

// schema/index.ts (Barrel File - Required)
export * from './users.schema';
export * from './orders.schema';
```

```typescript
// drizzle.config.ts (Solo: repo root | Monorepo: packages/db/drizzle.config.ts)
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  dialect: 'postgresql',
  schema: './src/db/schema/index.ts', // In packages/db: './src/schema/index.ts'
  out: './src/db/migrations',         // In packages/db: './migrations'
  dbCredentials: { url: process.env.DATABASE_URL! },
});
```

---

## 2. Drizzle DI Wiring: Solo Repo vs. Monorepo

### Option A: Solo Repo (`src/db/db.module.ts`)
```typescript
import { Global, Module } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { drizzle, NodePgDatabase } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import * as schema from './schema';

export const DB_PROVIDER = Symbol('DB_PROVIDER');
export type DrizzleDb = NodePgDatabase<typeof schema>;

@Global()
@Module({
  providers: [
    {
      provide: DB_PROVIDER,
      inject: [ConfigService],
      useFactory: (config: ConfigService) =>
        drizzle(new Pool({ connectionString: config.getOrThrow<string>('DATABASE_URL') }), { schema }),
    },
  ],
  exports: [DB_PROVIDER],
})
export class DbModule {}
```

### Option B: Monorepo (`packages/db` + `apps/api/src/db/db.module.ts`)
```typescript
// packages/db/src/index.ts (@repo/db)
import { drizzle, NodePgDatabase } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import * as schema from './schema';

export type DrizzleDb = NodePgDatabase<typeof schema>;
export const createDbClient = (url: string): DrizzleDb => drizzle(new Pool({ connectionString: url }), { schema });
export * from './schema';
export { eq, and, or, desc, asc, sql, ilike } from 'drizzle-orm';

// apps/api/src/db/db.module.ts wraps createDbClient() under DB_PROVIDER
```

---

## 3. External Auth Guard (Clerk / Supabase / Auth0) & User Webhook Sync

```typescript
// src/common/decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
export type Role = 'user' | 'admin';

export interface AuthUser {
  externalId: string; // Provider `sub`
  email: string;
  role: Role;
}

export const CurrentUser = createParamDecorator(
  (field: keyof AuthUser | undefined, ctx: ExecutionContext) => {
    const user = ctx.switchToHttp().getRequest<{ user?: AuthUser }>().user;
    return field ? user?.[field] : user;
  },
);
```

```typescript
// src/common/guards/external-auth.guard.ts
import { CanActivate, ExecutionContext, Injectable, UnauthorizedException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '../decorators/public.decorator';

@Injectable()
export class ExternalAuthGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    if (isPublic) return true;

    const req = context.switchToHttp().getRequest();
    const token = req.headers.authorization?.replace(/^Bearer\s+/i, '');
    if (!token) {
      throw new UnauthorizedException({ code: 'UNAUTHORIZED', message: 'Missing bearer token' });
    }

    try {
      // Verify token via Clerk verifyToken() / Supabase auth.getUser(token) / Auth0 JWKS
      const claims = await this.verifyProviderToken(token);
      req.user = {
        externalId: claims.sub,
        email: claims.email,
        role: (claims.metadata?.role as 'user' | 'admin') ?? 'user',
      };
      return true;
    } catch {
      throw new UnauthorizedException({ code: 'UNAUTHORIZED', message: 'Invalid or expired token' });
    }
  }

  private async verifyProviderToken(token: string): Promise<{ sub: string; email: string; metadata?: Record<string, unknown> }> {
    // Plug in @clerk/backend verifyToken, supabase.auth.getUser, or jose jwtVerify
    throw new Error('Implement provider SDK verification');
  }
}
```

### Idempotent Drizzle Upsert (Used by Auth Webhook & Fallback Sync)
```typescript
async upsertFromExternalProvider(input: { externalId: string; email: string; role?: string }) {
  const [user] = await this.db
    .insert(users)
    .values({
      externalId: input.externalId,
      email: input.email.toLowerCase().trim(),
      role: input.role ?? 'user',
    })
    .onConflictDoUpdate({
      target: users.externalId,
      set: { email: input.email.toLowerCase().trim(), updatedAt: new Date() },
    })
    .returning();
  return user;
}
```

---

## 4. Custom Decorators (`@Public`, `@Roles`, `@Auth`) & `CorrelationIdMiddleware`

```typescript
// src/common/decorators/public.decorator.ts
import { SetMetadata } from '@nestjs/common';
export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// src/common/decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';
import { Role } from './current-user.decorator';
export const ROLES_KEY = 'roles';
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);

// src/common/decorators/auth.decorator.ts
import { applyDecorators, UseGuards } from '@nestjs/common';
import { ExternalAuthGuard } from '../guards/external-auth.guard';
import { RolesGuard } from '../guards/roles.guard';
import { Role } from './current-user.decorator';
import { Roles } from './roles.decorator';

export function Auth(...roles: Role[]) {
  return applyDecorators(Roles(...roles), UseGuards(ExternalAuthGuard, RolesGuard));
}
```

```typescript
// src/common/middleware/correlation-id.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { randomUUID } from 'crypto';
import { NextFunction, Request, Response } from 'express';

@Injectable()
export class CorrelationIdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction): void {
    const requestId = (req.headers['x-request-id'] as string) || randomUUID();
    req.headers['x-request-id'] = requestId;
    res.setHeader('X-Request-Id', requestId);
    next();
  }
}
```
