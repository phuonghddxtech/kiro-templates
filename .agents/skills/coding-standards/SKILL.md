---
name: coding-standards
description: Mandatory coding standards covering clean code principles, code style, error handling, and naming conventions. Invoke when writing or reviewing any code to ensure quality and consistency.
---

# Coding Standards Skill

> This skill consolidates: clean-code, code-style, error-handling, and naming-conventions rules.

---

## 📦 Variables — Clean Code Rules

- Use **meaningful, pronounceable names**: `const currentDate` not `const yyyymmdstr`
- Same vocabulary for same type: `getUser()` not `getUserInfo()` / `getClientData()`
- **No magic numbers** — use named constants:
  ```js
  const MILLISECONDS_PER_DAY = 60 * 60 * 24 * 1000;
  setTimeout(blastOff, MILLISECONDS_PER_DAY);
  ```
- Avoid mental mapping — be explicit: `location` not `l`
- Don't add redundant context: `{ make, model, color }` not `{ carMake, carModel, carColor }`
- Use default parameters: `function create(name = 'Default') {}`

---

## 🔧 Functions

- **Max 2 arguments** — use object destructuring for more:
  ```js
  // ❌ function createMenu(title, body, buttonText, cancellable) {}
  // ✅ function createMenu({ title, body, buttonText, cancellable }) {}
  ```
- **Do ONE thing** per function
- Function names must say what they do: `addMonthToDate()` not `add()`
- **No flag parameters** — split into separate functions
- **No side effects** — don't mutate global state or arguments:
  ```js
  // ✅ Return new array instead of mutating
  function addItemToCart(cart, item) {
    return [...cart, { item, date: Date.now() }];
  }
  ```
- Favor `filter()`, `map()`, `reduce()` over for-loops with mutations
- Encapsulate conditionals: `if (shouldShowSpinner(fsm)) {}` not `if (fsm.state === 'fetching' && isEmpty(node)) {}`
- Avoid negative conditionals: `if (isDOMNodePresent(node))` not `if (!isNotDOMNodePresent(node))`
- Remove dead code immediately

---

## 🧱 SOLID Principles

| Principle | Rule |
|-----------|------|
| **S** — Single Responsibility | One class = one job |
| **O** — Open/Closed | Open for extension, closed for modification |
| **L** — Liskov Substitution | Subclasses substitutable for base class |
| **I** — Interface Segregation | Don't depend on interfaces you don't use |
| **D** — Dependency Inversion | Depend on abstractions, not concretions |

```js
// ✅ D — Dependency Inversion example
class InventoryService {
  constructor(inventoryRequester) { // inject the dependency
    this.inventoryRequester = inventoryRequester;
  }
}
```

---

## ⚡ Concurrency

- Use `async/await` over raw Promises/callbacks:
  ```js
  async function getCleanCodeArticle() {
    try {
      const response = await request.get(cleanCodeUrl);
      await fs.writeFile('article.html', response);
    } catch (err) {
      console.error(err);
    }
  }
  ```

---

## 📐 Code Style — Formatting

- Indentation: **2 spaces** (no tabs)
- Max line length: **100 characters**
- Use **single quotes** for strings
- Always use **semicolons**
- Trailing commas in multi-line structures
- One class/service per file
- Group related files in feature folders

### Naming Casing

```js
// Variables and functions: camelCase
const userProfile = {};
function getUserById(id) {}

// Classes and interfaces: PascalCase
class UserService {}

// Constants: UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;

// Files: kebab-case
// user-service.js, auth-middleware.js
```

### Import Order
```js
// 1. Node built-ins
import path from 'path';
// 2. External deps
import express from 'express';
// 3. Internal modules
import { UserService } from './user-service.js';
```

---

## 💥 Error Handling

### Custom Error Class
```js
// src/utils/app-error.js
class AppError extends Error {
  constructor(message, statusCode = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.name = 'AppError';
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}
```

### Throwing Errors
```js
if (!user) throw new AppError('User not found', 404, 'USER_NOT_FOUND');
if (!hasPermission) throw new AppError('Forbidden', 403, 'ACCESS_DENIED');
```

### Async Error Handling
```js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);
```

### Global Error Handler (Express)
```js
export function errorHandler(err, req, res, next) {
  const statusCode = err.statusCode || 500;
  logger.error({ err, req: { method: req.method, url: req.url } });

  if (!err.isOperational) {
    return res.status(500).json({
      success: false,
      error: { code: 'INTERNAL_ERROR', message: 'An unexpected error occurred' }
    });
  }

  res.status(statusCode).json({
    success: false,
    error: { code: err.code, message: err.message }
  });
}
```

**Core Rules:**
- **Never swallow errors silently** — always log or rethrow
- Use a centralized error handler
- Return consistent error responses
- Distinguish operational errors (expected) vs programmer errors (bugs)

---

## 🔑 Naming Conventions

### Cache Key Format
```
{app}:{version}:{entity}:{identifier}:{variant}

myapp:v1:user:12345:profile
myapp:v1:session:abc123xyz
myapp:v1:rate_limit:ip:192.168.1.1
myapp:v1:lock:payment:order:99999
```

### Cache TTL Conventions
| Data Type | TTL |
|-----------|-----|
| User session | 7 days |
| Auth tokens | 15 minutes |
| User profile | 1 hour |
| Config/settings | 24 hours |
| Rate limit windows | 15 minutes |

### Database Naming
```sql
-- Tables: snake_case, plural
users, order_items, product_categories

-- Columns: snake_case
id, user_id, created_at, is_active, has_verified_email

-- Indexes: idx_{table}_{columns}
idx_users_email
idx_orders_user_id_created_at

-- Foreign keys: fk_{child}_{parent}
fk_orders_users
```

### Queue/Event Naming
```
# Queue names (BullMQ): {app}.{entity}.{action}
myapp.email.send
myapp.payment.process

# Events (past tense): {entity}.{past_tense_verb}
user.registered
order.placed
payment.failed

# Dead Letter Queues
myapp.email.send.dlq
```

### Environment Variables
```bash
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://...
REDIS_URL=redis://localhost:6379
JWT_SECRET=...
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
BCRYPT_ROUNDS=12
```

### URL / Route Naming
```
GET    /api/v1/users
GET    /api/v1/users/:id
POST   /api/v1/users
PATCH  /api/v1/users/:id
DELETE /api/v1/users/:id

# Actions that don't fit CRUD
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
POST   /api/v1/payments/:id/refund
```

---

## 🗒️ Comments

```js
// ✅ Good — explains WHY, not WHAT
// We retry 3x because OAuth2 tokens can have clock skew
const MAX_RETRIES = 3;

// ❌ Never leave commented-out code
// ❌ No journal comments (use git log instead)
// ❌ No positional markers (/////...)
```
