---
description: Deploy the application to the target environment
---

## Steps

### 1. Pre-deploy Checklist
- Ensure all tests pass: `npm test`
- Check for linting errors: `npm run lint`
- Confirm environment variables are configured
- Confirm database migrations are ready

// turbo
### 2. Verify Clean Git State
```bash
git status
git pull origin main
```

// turbo
### 3. Run Tests
```bash
npm test
```
If tests fail → STOP and report failures.

// turbo
### 4. Build
```bash
npm run build
```

### 5. Deploy by Environment

**Development:**
```bash
docker-compose -f docker-compose.dev.yml up -d --build
# or: npm run deploy:dev
```

**Production:**
```bash
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d
# or: npm run deploy:prod
```

// turbo
### 6. Post-deploy Health Check
```bash
curl -f http://localhost:3000/health || echo "Health check failed"
```

// turbo
### 7. Tail Logs (first 30 seconds)
```bash
docker-compose logs --tail=50 -f app
```

### 8. Post-deploy Verification
- Check application health endpoint returns 200
- Verify logs for errors
- Run smoke tests

## Rollback
If deployment fails:
```bash
git revert HEAD --no-edit
git push origin main
npm run rollback
```
