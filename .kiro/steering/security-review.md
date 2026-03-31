# Security Review Workflow

When asked to perform a security audit, follow this checklist:

## 🔴 Critical (Check First)
- [ ] Hardcoded secrets, API keys, passwords in source files
  ```bash
  grep -r "password\s*=\s*['\"]" src/
  grep -r "api_key\s*=\s*['\"]" src/
  ```
- [ ] `.env` files accidentally committed
  ```bash
  git log --all --full-history -- .env
  ```
- [ ] SQL injection via string concatenation
- [ ] `eval()` or `new Function()` with user input

## 🟡 High Priority
- [ ] Missing authentication on protected routes
- [ ] Missing authorization (privilege escalation)
- [ ] Passwords stored in plain text
- [ ] JWT secrets too short or exposed
- [ ] No rate limiting on auth endpoints
- [ ] Missing input validation

## 🟢 Medium Priority
- [ ] Missing security headers (run Helmet scan)
- [ ] CORS configured too broadly (`origin: *`)
- [ ] Dependencies with known vulnerabilities: `npm audit`
- [ ] Sensitive data in logs
- [ ] Missing HTTPS enforcement

## ℹ️ Low / Informational
- [ ] Error messages revealing stack traces to client
- [ ] Missing CSP headers
- [ ] Cookie security flags (HttpOnly, Secure, SameSite)

## Output Format
```markdown
# Security Review Report — [Date]

## Critical Issues
[List with file:line references]

## High Priority Issues
[List with file:line references]

## Recommendations
[Prioritized action items]
```

## Commands
```bash
npm audit --audit-level=moderate

grep -rn --include="*.js" --include="*.ts" \
  -E "(password|secret|api_key|token)\s*=\s*['\"][^'\"]{8,}" src/
```
