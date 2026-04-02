---
name: security
description: CRITICAL security requirements that must NEVER be violated. Invoke when building any feature that handles user data, authentication, authorization, input, or external services.
---

# Security Skill

> ⚠️ CRITICAL — These rules must NEVER be violated under any circumstances.

## 🚨 CRITICAL — Never Violate These

- **NEVER** hardcode secrets, API keys, passwords, or tokens in source code
- **NEVER** commit `.env` files to version control
- **NEVER** log sensitive data (passwords, tokens, PII, credit cards)
- **NEVER** use `eval()` or `Function()` with user input
- **ALWAYS** validate and sanitize ALL user inputs

---

## Environment Variables
```js
// ✅ Always use environment variables for secrets
const dbPassword = process.env.DB_PASSWORD;
const jwtSecret = process.env.JWT_SECRET;

// ❌ Never hardcode secrets
const dbPassword = 'mypassword123'; // FORBIDDEN
```

---

## Input Validation
```js
// ✅ Validate all incoming data with a schema (Zod)
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email().max(255),
  password: z.string().min(8).max(128)
});

// Sanitize HTML to prevent XSS
import DOMPurify from 'dompurify';
const cleanContent = DOMPurify.sanitize(userInput);
```

---

## Authentication
- Use **JWT** with short expiry: 15 min access token, 7 day refresh token
- Hash passwords with **bcrypt** (rounds: 12+)
- Implement rate limiting on auth endpoints

```js
import bcrypt from 'bcrypt';
const SALT_ROUNDS = 12;
const hashed = await bcrypt.hash(password, SALT_ROUNDS);
```

---

## Authorization
```js
// ✅ Check permissions on every protected route
router.delete('/posts/:id', authenticate, authorize('admin'), asyncHandler(deletePost));

// ✅ Verify resource ownership
if (post.authorId !== req.user.id && req.user.role !== 'admin') {
  throw new AppError('Forbidden', 403);
}
```

---

## HTTP Security Headers
```js
// Use Helmet.js
import helmet from 'helmet';
app.use(helmet());

// Configure CORS strictly — never use origin: *
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || [],
  credentials: true
}));
```

---

## Rate Limiting
```js
import rateLimit from 'express-rate-limit';

const apiLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 100 });
const authLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 5 }); // stricter for auth

app.use('/api/', apiLimiter);
app.use('/api/auth/', authLimiter);
```

---

## SQL Injection Prevention
- Always use ORM parameterized queries (Prisma)
- **NEVER** concatenate user input into SQL strings

---

## Dependency Security
```bash
# Regularly audit dependencies
npm audit
npm audit fix
```

---

## Security Review Quick Checklist
```markdown
### 🔴 Critical (Check First)
- [ ] No hardcoded secrets or API keys in source files
  grep -r "password\s*=\s*['\"]" src/
- [ ] .env files not committed: git log --all --full-history -- .env
- [ ] No SQL injection via string concatenation
- [ ] No eval() or new Function() with user input

### 🟡 High Priority
- [ ] All protected routes have authentication middleware
- [ ] Resource ownership verified (can user A access user B's data?)
- [ ] Passwords stored hashed (bcrypt, not MD5/SHA1)
- [ ] JWT secrets are long and random
- [ ] Rate limiting on auth endpoints
- [ ] All inputs validated with schema

### 🟢 Medium Priority
- [ ] Security headers (Helmet.js)
- [ ] CORS not configured too broadly
- [ ] npm audit clean
- [ ] No sensitive data in logs
- [ ] HTTPS enforced in production
```
