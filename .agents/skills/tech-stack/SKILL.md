---
name: tech-stack
description: Approved technology stack and architecture standards for all projects. Invoke when starting a new project, choosing a library, or making technology decisions.
---

# Tech Stack Skill

> Defines the **approved tech stack** for all projects. When proposing a new dependency, evaluate against the decision criteria below first.

---

## 🗂️ Approved Stack — Quick Reference

| Layer | Primary Choice | Alternative | Avoid |
|-------|---------------|-------------|-------|
| **Frontend — Landing/SEO** | Next.js 14+ (App Router) | — | CRA (deprecated) |
| **Frontend — Admin/Dashboard** | React + Vite (SPA) | — | Next.js (overkill) |
| **UI Components** | shadcn/ui + Radix UI | Chakra UI | MUI (too heavy) |
| **Styling** | Tailwind CSS | CSS Modules | Styled-components |
| **State Management** | Zustand | Redux Toolkit | MobX, Recoil |
| **Data Fetching** | TanStack Query (React Query) | SWR | Axios alone |
| **Backend Framework** | Express.js + Node | Fastify | Hapi, Koa |
| **API Style** | REST (default) | tRPC (fullstack TS) | GraphQL (unless needed) |
| **Language** | TypeScript (always) | — | Plain JavaScript |
| **Database** | PostgreSQL | — | MySQL (prefer PG) |
| **ORM** | Prisma | Drizzle | Sequelize, TypeORM |
| **Cache** | Redis (ioredis) | Upstash Redis | Memcached |
| **Queue — Simple** | BullMQ (Redis-backed) | — | — |
| **Queue — Enterprise** | RabbitMQ | Kafka (high-throughput) | — |
| **Auth** | NextAuth.js (Next) / JWT+bcrypt (API) | Lucia Auth | Firebase Auth |
| **File Storage** | AWS S3 / Cloudflare R2 | Supabase Storage | Local disk |
| **Email** | Resend | Nodemailer + SMTP | SendGrid |
| **Search** | PostgreSQL FTS (start here) | Meilisearch | Elasticsearch |
| **Monitoring** | Grafana + Prometheus | Datadog | — |
| **Logging** | Pino | Winston | console.log (production) |
| **Testing** | Vitest + Testing Library | Jest | Mocha |
| **E2E Testing** | Playwright | Cypress | Selenium |
| **CI/CD** | GitHub Actions | — | Jenkins |
| **Containerization** | Docker + Docker Compose | — | — |
| **Deployment** | Vercel (frontend) + Railway/Fly.io (backend) | AWS | — |
| **API Docs** | Swagger / OpenAPI 3.0 | — | Postman only |

---

## 🖥️ Frontend — Choosing the Right Framework

> **Rule**: A project typically has **both** — Next.js for public site + React for admin.

### Next.js 14 — Landing Page / SEO
Use when: **SEO matters**, public-facing pages, marketing sites, blogs

```bash
npx create-next-app@latest my-landing \
  --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
```

**Why Next.js:**
- Server-Side Rendering (SSR) → Google can crawl content
- Static Site Generation (SSG) → HTML on CDN, very fast
- Automatic image optimization (`next/image`)
- Integrated metadata API for SEO

```tsx
// app/layout.tsx — SEO metadata
export const metadata: Metadata = {
  title: { default: 'My App', template: '%s | My App' },
  description: 'Main page description',
  openGraph: { type: 'website', locale: 'en_US', url: 'https://myapp.com' },
};
```

**Folder structure (App Router)**
```
src/app/
├── (marketing)/          # Public pages (SSG/SSR)
│   ├── page.tsx          # Homepage
│   ├── about/page.tsx
│   └── pricing/page.tsx
├── (auth)/               # Auth pages
│   ├── login/page.tsx
│   └── register/page.tsx
├── api/v1/               # API Routes
└── layout.tsx
```

### React + Vite — Admin / Dashboard
Use when: **No SEO needed**, admin panels, internal tools
```bash
npx create-vite@latest my-admin -- --template react-ts
```

---

## 🗄️ Database — PostgreSQL + Prisma

### Prisma Schema Conventions
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

### Migration Workflow
```bash
npx prisma migrate dev --name add_user_role  # Development
npx prisma migrate deploy                     # Production
npx prisma studio                             # GUI browser
```

---

## ⚡ Cache — Redis (ioredis)

### Redis Client Setup
```ts
// src/lib/redis.ts
import Redis from 'ioredis';

export const redis = new Redis(process.env.REDIS_URL!, {
  maxRetriesPerRequest: 3,
  enableReadyCheck: true,
  lazyConnect: true,
});
```

### Cache Helper Pattern
```ts
export async function getOrSet<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttlSeconds = 3600
): Promise<T> {
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  const data = await fetcher();
  await redis.setex(key, ttlSeconds, JSON.stringify(data));
  return data;
}

export async function invalidate(pattern: string) {
  const keys = await redis.keys(pattern);
  if (keys.length) await redis.del(...keys);
}
```

---

## 📨 Queue — Choosing The Right Type

| Criterion | BullMQ | RabbitMQ | Kafka |
|-----------|--------|----------|-------|
| **Use when** | Simple jobs, retry, schedule | Microservices, complex routing | Event streaming, billions msgs |
| **Throughput** | Medium | High | Extreme (millions/s) |
| **Replay** | ❌ | ❌ | ✅ |
| **Complexity** | Low | Medium | High |

> **Rule**: Default to **BullMQ**. Use RabbitMQ for microservices. Use Kafka only for streaming or replay.

```ts
// BullMQ (default)
const emailQueue = new Queue('email', {
  connection: redis,
  defaultJobOptions: { attempts: 3, backoff: { type: 'exponential', delay: 2000 } },
});
```

---

## ✅ Technology Decision Process

When proposing a new library, evaluate:

| Criterion | Questions |
|-----------|-----------| 
| **Necessity** | Does an approved alternative already solve this? |
| **Maintenance** | Stars > 1k? Last commit < 6 months? |
| **Bundle size** | Check bundlephobia.com |
| **TypeScript** | Native TS types? |
| **License** | MIT/Apache? (No GPL in commercial) |
| **Security** | `npm audit` — zero high/critical vulnerabilities? |

### Decision Template
```markdown
## Technology Decision: [Library Name]

**Problem**: What problem does this solve?
**Alternative evaluated**: What from the approved stack was considered?
**Why chosen**: Specific reason this is better for the use case
**Risk**: Known downsides or migration cost
**Decision**: ✅ Adopt / ❌ Reject
```
