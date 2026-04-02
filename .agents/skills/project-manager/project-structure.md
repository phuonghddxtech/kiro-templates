---
name: project-manager
description: Strategic project manager who plans sprints, defines requirements, tracks progress, and ensures delivery. Invoke when planning features, breaking down tasks, writing user stories, or reviewing project status.
---

# Project Structure & Architecture Standards

> This document defines the standard folder layout, layered architecture, and API conventions for all projects.

---

## Standard Folder Layout

```
project-root/
├── .agents/                    # Antigravity AI Agent configuration
│   ├── skills/                 # Skill definitions
│   ├── workflows/              # Workflow commands
│   └── ...
│
├── src/                        # Application source code
│   ├── config/                 # Configuration files
│   ├── controllers/            # Route handlers (thin layer)
│   ├── middleware/             # Express middleware
│   ├── models/                 # Database models/schemas
│   ├── repositories/           # Data access layer
│   ├── routes/                 # Route definitions
│   ├── services/               # Business logic layer
│   ├── utils/                  # Utility functions
│   └── index.js                # Application entry point
│
├── tests/                      # Test files
│   ├── unit/                   # Unit tests
│   ├── integration/            # Integration tests
│   └── e2e/                    # End-to-end tests
│
├── docs/                       # Documentation
│   ├── api/                    # API documentation
│   └── architecture/           # Architecture diagrams & ADRs
│
├── scripts/                    # Build and utility scripts
├── .env.example                # Example environment variables
├── .gitignore
├── package.json
└── README.md
```

## Layered Architecture
```
Request → Routes → Middleware → Controllers → Services → Repositories → Database
```

- **Routes**: URL mapping only, no logic
- **Controllers**: Request/response handling, input validation
- **Services**: Business logic, orchestration
- **Repositories**: Data access, queries
- **Models**: Data schemas and types

## File Naming
- Source files: `kebab-case.js` (`user-service.js`)
- Test files: `[name].test.js` (`user-service.test.js`)
- Config files: `kebab-case.js` or `kebab-case.json`

## Environment Files
- `.env` — Local development (gitignored)
- `.env.example` — Template committed to git
- `.env.test` — Test environment (gitignored)
- `.env.production` — Set in CI/CD, never committed

---

## REST API Design Standards

### URL Structure
- Use **kebab-case** for URL paths: `/api/user-profiles`
- Use **plural nouns** for resource collections: `/api/users`, `/api/products`
- Nest related resources: `/api/users/:id/orders`
- API version prefix: `/api/v1/...`

### HTTP Methods
| Method | Usage |
|--------|-------|
| GET | Read resources (idempotent) |
| POST | Create new resource |
| PUT | Replace entire resource |
| PATCH | Partial update |
| DELETE | Remove resource |

### HTTP Status Codes
| Code | Meaning |
|------|---------|
| 200 | OK — Successful GET/PUT/PATCH |
| 201 | Created — Successful POST |
| 204 | No Content — Successful DELETE |
| 400 | Bad Request — Invalid input |
| 401 | Unauthorized — Not authenticated |
| 403 | Forbidden — No permission |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Unprocessable Entity — Validation failed |
| 500 | Internal Server Error |

### Request/Response Format
```json
// Success response
{ "success": true, "data": { ... }, "message": "Optional message" }

// Error response
{ "success": false, "error": { "code": "VALIDATION_ERROR", "message": "..." } }

// Paginated list response
{
  "success": true,
  "data": [ ... ],
  "pagination": { "page": 1, "limit": 20, "total": 100, "totalPages": 5 }
}
```

### Filtering & Pagination
```
GET /api/users?page=1&limit=20&sortBy=createdAt&order=desc
GET /api/products?category=electronics&minPrice=100
```

### Documentation
- Every endpoint **MUST** have JSDoc/Swagger annotations
- Include request body schema, response schema, and error codes

---

## Database Rules

- **Never** write raw SQL strings in business logic — always use ORM (Prisma)
- All database calls must be inside try/catch blocks
- Use **transactions** for multi-step operations

```js
// ✅ Select only needed fields
const user = await db.user.findUnique({
  where: { id },
  select: { id: true, email: true, name: true }
});

// ✅ Use pagination for lists
const users = await db.user.findMany({
  take: limit,
  skip: (page - 1) * limit,
  orderBy: { createdAt: 'desc' }
});

// ✅ Transactions for atomic operations
await db.$transaction(async (tx) => {
  const order = await tx.order.create({ data: orderData });
  await tx.inventory.update({ where: { id: productId }, data: { stock: { decrement: 1 } } });
  return order;
});
```

### Migration Rules
- Always use migration files — never modify schema directly
- Migration files are version-controlled and **immutable**
- Run migrations in CI/CD before deploying

---

## Testing Standards

### Testing Pyramid
```
     [E2E Tests]         ← Few, slow, catch integration issues
   [Integration Tests]   ← Some, test component interaction
 [Unit Tests]            ← Many, fast, test isolated logic
```

### Requirements
- Unit test coverage: **minimum 80%**
- All new features must have tests
- All bug fixes must have a **regression test**
- Tests run in CI before any merge

### Test Commands
```bash
npm test                    # Run all tests
npm run test:unit           # Unit tests only
npm run test:integration    # Integration tests only
npm run test:coverage       # Generate coverage report
npm run test:watch          # Watch mode
```

### Naming Conventions
- Test files: `[filename].test.js`
- `describe` blocks: Match the module/function being tested
- `it` blocks: `should [expected behavior] when [condition]`
