---
description: Upgrade Expo SDK to a new version safely
---

## Steps

### 1. Check Current Version
// turbo
```bash
npx expo --version
cat package.json | grep expo
```

### 2. Read Upgrade Guide
- Check [Expo SDK changelog](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/)
- Identify breaking changes for your SDK version jump
- List affected dependencies

### 3. Create Upgrade Branch
```bash
git checkout -b chore/upgrade-expo-sdk-{version}
```

### 4. Run Expo Upgrade Command
```bash
npx expo install expo@latest
```
This will also upgrade compatible dependencies.

### 5. Update Additional Dependencies
// turbo
```bash
npx expo install --fix
```
This fixes any version mismatches.

### 6. Check for Breaking Changes
Review each dependency:
- [ ] `expo-router` — check for API changes
- [ ] `expo-image` — check for prop changes
- [ ] `expo-notifications` — check for config changes
- [ ] `expo-secure-store` — check for API changes
- [ ] `react-native-reanimated` — check for compatibility
- [ ] `react-native-gesture-handler` — check for compatibility

### 7. Update app.config.ts
```ts
// Verify SDK version matches
export default {
  expo: {
    sdkVersion: '{newVersion}',
  },
};
```

### 8. Clear Cache & Reinstall
```bash
rm -rf node_modules
rm package-lock.json
npm install
npx expo start --clear
```

### 9. Test Both Platforms
// turbo
```bash
npm test -- --watchAll=false
```

- [ ] App builds on iOS simulator
- [ ] App builds on Android emulator
- [ ] All screens render correctly
- [ ] Navigation works
- [ ] API calls work
- [ ] Push notifications work
- [ ] Animations work

### 10. Update EAS Build Config
If `eas.json` has SDK-specific settings, update them.

### 11. Commit
```
chore: upgrade expo sdk to {version}

- Updated expo and all dependencies
- Fixed breaking changes in {list}
- Tested on iOS and Android
```

## Rollback
If upgrade causes critical issues:
```bash
git checkout main
git branch -D chore/upgrade-expo-sdk-{version}
```
