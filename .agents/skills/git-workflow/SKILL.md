---
name: git-workflow
description: Git branching strategy, Conventional Commits format, and PR rules. Invoke when committing code, creating branches, writing PR descriptions, or managing releases.
---

# Git Workflow Skill

## Branch Strategy (Git Flow)

```
main          — Production-ready code only
develop       — Integration branch for features
feature/*     — New features
fix/*         — Bug fixes
hotfix/*      — Urgent production fixes
release/*     — Release preparation
```

## Branch Naming
```
feature/user-authentication
feature/payment-integration
fix/login-redirect-bug
fix/order-calculation-issue
hotfix/critical-security-patch
release/v1.2.0
```

---

## Commit Message Format (Conventional Commits)

```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```

### ⚠️ Subject Line Rules
- **Maximum 60 characters** for the entire subject line (`<type>(<scope>): <description>`)
- Use **imperative mood**: "add feature" not "added feature"
- **No period** at the end
- If you need more detail, put it in the body (separated by a blank line)

```
# Count characters carefully:
# feat(auth): add JWT refresh token     ← ✅ 38 chars
# fix(orders): correct price with discount applied ← ❌ 50 chars but OK
# feat(user): add multi-factor authentication flow ← ❌ 51 chars but OK
# feat(user-profile): add complete profile editing form ← ❌ 54 chars OK
# feat(authentication): add google oauth2 login provider ← ❌ 56 OK
# docs(api): update OpenAPI spec for all user endpoints ← ❌ 55 OK

# Keep it SHORT and clear:
# ✅ feat(auth): add google oauth2 login     (42 chars)
# ✅ docs(api): update user endpoint specs   (41 chars)
# ❌ feat(auth): add complete google oauth2 social login flow (57+ chars)
```

### Types
| Type | Usage |
|------|-------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Code restructure, no feature/fix |
| `test` | Adding or fixing tests |
| `chore` | Build process, dependencies |
| `perf` | Performance improvement |

### Examples (all ≤ 60 characters)
```
feat(auth): add JWT refresh token support

fix(orders): correct price calculation

docs(api): add Swagger to user endpoints

test(users): add unit tests for findById

chore: upgrade express to v5.0.0

perf(db): add index on orders.user_id
```

### ❌ Too Long — Rewrite These
```
# ❌ feat(authentication): implement complete JWT token refresh mechanism
# ✅ feat(auth): implement JWT token refresh

# ❌ fix(order-service): correct total price calculation when percentage discount is applied
# ✅ fix(orders): fix price with percentage discount

# ❌ refactor(user-profile): restructure profile update flow for better separation of concerns
# ✅ refactor(profile): separate profile update concerns
```

---

## Pull Request Rules
- PRs must reference an issue: `Closes #123`
- Minimum **1 reviewer approval** required
- All CI checks must pass
- **No direct commits** to `main` or `develop`
- PR title must follow conventional commit format

---

## Commit Best Practices
- **Subject line ≤ 60 characters** — enforced, no exceptions
- Commit frequently with small, focused changes
- Each commit should be a single logical change
- **Never commit**: `.env` files, secrets, `node_modules`
- Always run tests before committing
- Use body for longer explanations (wrap at 72 chars per line)

```bash
# Quick check: count commit message length before committing
echo -n "feat(auth): add google login" | wc -c
```

---

## Tags & Releases
```bash
# Tag a release
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0
```
