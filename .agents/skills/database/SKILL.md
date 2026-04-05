---
name: database
description: Database best practices covering query patterns, transactions, migrations, connection management, and security. Invoke when writing queries, designing schemas, running migrations, or reviewing database code.
---

# Database Skill

> Database rules for PostgreSQL + Prisma projects.

## General Rules
- **Never** write raw SQL strings directly in business logic
- Always use an ORM (Prisma) or query builder
- All database calls must be inside try/catch blocks
- Use **transactions** for multi-step operations

---

## Connection Management
```js
// ✅ Use connection pooling
const pool = new Pool({ max: 10, idleTimeoutMillis: 30000 });

// ❌ Never create a new connection per request
```

### Prisma Client — Singleton Pattern
```ts
// src/lib/db.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const db =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = db;
```

---

## Query Best Practices
```js
// ✅ Select only needed fields
const user = await db.user.findUnique({
  where: { id },
  select: { id: true, email: true, name: true }
});

// ❌ Avoid SELECT * — don't fetch unnecessary data
const user = await db.user.findUnique({ where: { id } });

// ✅ Use pagination for lists
const users = await db.user.findMany({
  take: limit,
  skip: (page - 1) * limit,
  orderBy: { createdAt: 'desc' }
});
```

### N+1 Query Prevention
```js
// ❌ N+1: one query per user
const users = await db.users.findAll();
for (const user of users) {
  user.orders = await db.orders.findAll({ where: { userId: user.id } });
}

// ✅ Single query with include
const users = await db.user.findMany({
  include: { orders: true }
});
```

---

## Transactions
```js
// ✅ Use transactions for atomic operations
await db.$transaction(async (tx) => {
  const order = await tx.order.create({ data: orderData });
  await tx.inventory.update({
    where: { id: productId },
    data: { stock: { decrement: 1 } }
  });
  return order;
});
```

---

## Migrations
- Always use migration files, never modify the database schema directly
- Migration files are version-controlled and immutable
- Run migrations in CI/CD before deploying

```bash
# Development: auto-migrate
npx prisma migrate dev --name add_user_role

# Production: apply pending migrations
npx prisma migrate deploy

# View DB in browser
npx prisma studio
```

---

## Prisma Schema Conventions
```prisma
model User {
  id        String   @id @default(cuid())   // cuid() for distributed systems
  email     String   @unique
  name      String?
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  deletedAt DateTime?                        // soft delete

  orders    Order[]

  @@map("users")                             // snake_case table name
  @@index([email])
}

enum Role {
  USER
  ADMIN
}
```

---

## Naming Conventions
- Tables: **snake_case** plural (`user_profiles`, `order_items`)
- Columns: **snake_case** (`created_at`, `user_id`)
- Booleans: `is_`, `has_`, `can_` prefix (`is_active`, `has_verified_email`)
- Foreign keys: `{referenced_table_singular}_id`
- Timestamps: `{event}_at` (`created_at`, `deleted_at`)
- Indexes: `idx_{table}_{columns}` (`idx_users_email`)
- Unique indexes: `uniq_{table}_{columns}` (`uniq_users_email`)
- Foreign key constraints: `fk_{child_table}_{parent_table}`

---

## PostgreSQL Best Practices
- Use `cuid()` or `uuid()` for primary keys (not auto-increment for distributed systems)
- Always add `@@index` on foreign keys and frequently queried columns
- Use `jsonb` columns for flexible/schema-less data instead of adding MongoDB
- Enable `pg_trgm` extension for fuzzy search
- Set `statement_timeout` and `lock_timeout` for long queries

---

## Indexing Rules
```sql
-- ✅ Index columns used in WHERE, JOIN, ORDER BY
CREATE INDEX idx_users_email ON users(email);

-- ❌ Don't index every column — indexes slow down writes
```

---

## Security
- Never log query results containing sensitive data (passwords, tokens)
- Use parameterized queries — never string concatenation
- Apply row-level security where applicable
