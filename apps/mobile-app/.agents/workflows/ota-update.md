---
description: Push an OTA update via EAS Update without app store review
---

## Steps

### 1. Verify Change is JS-only
OTA updates can ONLY include JavaScript/TypeScript changes.
**Cannot update**: native modules, `app.config.ts`, splash screen, icons.

If native code changed → use `/eas-deploy` workflow instead.

// turbo
### 2. Run Tests
```bash
npm test -- --watchAll=false
```
If tests fail → STOP.

// turbo
### 3. Verify Git State
```bash
git status
```
Ensure all changes are committed.

### 4. Push OTA Update
```bash
eas update --branch production --message "{description of fix}"
```

For staging/preview:
```bash
eas update --branch preview --message "{description}"
```

### 5. Verify on Device
- Open app → close → reopen (update downloads on next launch)
- Or use `expo-updates` API to check for updates programmatically:
```tsx
import * as Updates from 'expo-updates';

const update = await Updates.checkForUpdateAsync();
if (update.isAvailable) {
  await Updates.fetchUpdateAsync();
  await Updates.reloadAsync();
}
```

### 6. Monitor
- Check Sentry for new errors after rollout
- Check structured logs for anomalies
- Verify the fix is working on both iOS and Android

## Rollback
If the OTA update causes issues:
```bash
eas update --branch production --message "Rollback: {reason}"
```
This pushes a new update that reverts the previous change.
