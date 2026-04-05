---
description: Add a new native module or Expo plugin to the project
---

## Steps

### 1. Evaluate the Dependency
Before adding, answer these questions:
- [ ] **Necessity**: Can this be done without a new dependency?
- [ ] **Maintenance**: Is the package actively maintained? (last commit < 6 months)
- [ ] **Compatibility**: Does it support the current Expo SDK version?
- [ ] **TypeScript**: Does it have TypeScript types?
- [ ] **Bundle size**: What's the size impact? (check bundlephobia.com)
- [ ] **License**: Is the license compatible (MIT, Apache-2.0)?

If the package fails any check → find an alternative or discuss with team.

### 2. Check Expo Compatibility
// turbo
```bash
npx expo install {package-name} --check
```
If not compatible with managed workflow:
- Check if there's an Expo equivalent (e.g., `expo-camera` instead of `react-native-camera`)
- If no Expo equivalent → may need Expo Development Build

### 3. Install
```bash
# Expo-managed package
npx expo install {package-name}

# npm package (non-native)
npm install {package-name}
```

### 4. Configure Plugin (if native)
If the package requires native config, add to `app.config.ts`:
```ts
export default {
  expo: {
    plugins: [
      ['{package-name}', {
        // plugin options
      }],
    ],
  },
};
```

### 5. Check Platform Requirements

**iOS:**
- [ ] `Info.plist` permissions needed? (camera, location, etc.)
- [ ] Minimum iOS version compatible?

**Android:**
- [ ] `AndroidManifest.xml` permissions needed?
- [ ] Minimum Android SDK version compatible?

Add permissions via config plugin:
```ts
// app.config.ts
plugins: [
  ['expo-camera', { cameraPermission: 'Cho phép truy cập camera để chụp ảnh.' }],
],
```

### 6. Create Dev Build (if native module)
Native modules require a development build (not Expo Go):
```bash
eas build --profile development --platform all
```

### 7. Test on Both Platforms
- [ ] Feature works on **iOS**
- [ ] Feature works on **Android**
- [ ] No crash on permission denied
- [ ] Graceful fallback if feature unavailable

// turbo
### 8. Run Tests
```bash
npm test -- --watchAll=false
```

### 9. Document
Add a note in the PR about:
- Why this dependency was added
- What it's used for
- Any platform-specific setup required

### 10. Commit
```
feat({scope}): add {package-name} for {purpose}

- Installed {package-name}@{version}
- Configured plugin in app.config.ts
- Added permissions for iOS/Android
- Tested on both platforms
```
