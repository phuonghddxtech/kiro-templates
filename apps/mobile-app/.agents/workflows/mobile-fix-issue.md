---
description: Analyze and fix a mobile bug systematically on both iOS and Android
---

## Steps

### 1. Identify Platform Scope
Determine which platforms are affected:
- **iOS only** — likely SafeArea, keyboard, shadow, StatusBar issue
- **Android only** — likely elevation, BackHandler, font rendering issue
- **Both platforms** — likely logic, API, or state issue

### 2. Reproduce the Bug
- Run on iOS Simulator and/or Android Emulator
- Reproduce exact steps from bug report
- Check Sentry/crash reports for stack trace

// turbo
### 3. Check Recent Changes
```bash
git log --oneline -20
```
- Any recent commits that could have caused this?

### 4. Check Logs
- Review structured logs (`logger.error`, `logger.warn`)
- Check Sentry for error details
- Check API response interceptor logs

### 5. Root Cause Analysis
- Identify the affected component(s), hook(s), or service(s)
- Check for common mobile issues:
  - [ ] Missing null/undefined check
  - [ ] Race condition in async operations
  - [ ] Platform-specific behavior not handled
  - [ ] Missing error state handling
  - [ ] Stale closure in useCallback/useEffect
  - [ ] Missing `enabled` flag in TanStack Query

### 6. Implement Fix
- Make the minimal targeted fix
- Follow rules in `.agents/rules/error-handling.md`
- Add proper error handling if missing

### 7. Add Regression Test
Write a test that would have caught the bug:
```tsx
it('should handle null order gracefully', () => {
  // This test prevents the bug from happening again
});
```

// turbo
### 8. Run Tests on Both Platforms
```bash
npm test -- --watchAll=false
```

### 9. Manual Verification
- [ ] Bug is fixed on **iOS**
- [ ] Bug is fixed on **Android**
- [ ] No regression in related features
- [ ] Error states handled correctly

### 10. Commit
```
fix({scope}): {short description}

Closes #{issue-number}
```
