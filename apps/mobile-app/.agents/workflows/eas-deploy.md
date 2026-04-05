---
description: Build and deploy app via EAS Build and EAS Submit
---

## Steps

// turbo
### 1. Verify Clean Git State
```bash
git status
git pull origin main
```
If dirty → commit or stash changes first.

// turbo
### 2. Run Tests & Lint
```bash
npm test -- --watchAll=false
npm run lint
```
If tests fail → STOP and fix before deploying.

### 3. Bump Version (if releasing)
Update version in `app.config.ts` or `app.json`:
```bash
# Patch: 1.0.0 → 1.0.1
# Minor: 1.0.0 → 1.1.0
# Major: 1.0.0 → 2.0.0
```

### 4. Build with EAS
Choose the correct profile:

**Development (internal testing):**
```bash
eas build --profile development --platform all
```

**Preview (beta testers):**
```bash
eas build --profile preview --platform all
```

**Production (store release):**
```bash
eas build --profile production --platform all
```

**Single platform:**
```bash
eas build --profile production --platform ios
eas build --profile production --platform android
```

### 5. Submit to Stores (production only)
```bash
eas submit --platform ios
eas submit --platform android
```

### 6. Tag Release in Git
```bash
git tag -a v{version} -m "Release v{version}"
git push origin v{version}
```

### 7. Post-deploy Checklist
- [ ] App installs and launches on iOS
- [ ] App installs and launches on Android
- [ ] Critical flows work (login, main feature)
- [ ] Push notifications receive correctly
- [ ] Crash reporting (Sentry) is active
- [ ] Team notified of release

## Rollback
If critical issue found post-release:
```bash
# OTA rollback (JS-only issues)
eas update --branch production --message "Rollback: {reason}"

# Full rollback (native issues)
# Revert to previous build and re-submit
```
