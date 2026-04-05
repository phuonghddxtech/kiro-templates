---
description: Initialize a new React Native (Expo) project with all conventions and tooling
---

## Steps

### 1. Create Expo Project
```bash
npx create-expo-app@latest {project-name} --template blank-typescript
cd {project-name}
```

### 2. Install Core Dependencies
```bash
# Navigation
npx expo install expo-router react-native-screens react-native-safe-area-context

# State management
npm install zustand @tanstack/react-query

# Forms
npm install react-hook-form @hookform/resolvers zod

# HTTP
npm install axios

# Storage
npm install react-native-mmkv
npx expo install expo-secure-store

# Images
npx expo install expo-image

# Icons
npm install lucide-react-native react-native-svg

# Animation
npx expo install react-native-reanimated react-native-gesture-handler

# i18n
npx expo install expo-localization
npm install i18next react-i18next

# Notifications
npx expo install expo-notifications expo-device expo-constants

# Crash reporting
npx expo install @sentry/react-native

# Dev tools
npm install -D @testing-library/react-native jest @types/jest
```

### 3. Create Folder Structure
```
mkdir -p src/{app,components/{ui,feedback},hooks,services,stores,utils,constants,types,i18n/locales}
mkdir -p __tests__/{unit/components,integration/services,e2e}
```

Create core files:
```
src/
├── app/_layout.tsx
├── app/(auth)/_layout.tsx
├── app/(auth)/login.tsx
├── app/(tabs)/_layout.tsx
├── app/(tabs)/index.tsx
├── app/+not-found.tsx
├── components/ErrorBoundary.tsx
├── components/feedback/LoadingState.tsx
├── components/feedback/ErrorState.tsx
├── components/feedback/EmptyState.tsx
├── services/api.ts
├── stores/auth.store.ts
├── stores/app.store.ts
├── utils/logger.ts
├── utils/error.ts
├── utils/format.ts
├── constants/colors.ts
├── constants/spacing.ts
├── constants/config.ts
├── i18n/index.ts
├── i18n/locales/vi.json
└── i18n/locales/en.json
```

### 4. Setup Config Files

**app.config.ts:**
```ts
export default {
  expo: {
    name: '{project-name}',
    scheme: '{project-scheme}',
    plugins: ['expo-router', 'expo-secure-store'],
  },
};
```

**tsconfig.json** — add path aliases:
```json
{
  "compilerOptions": {
    "paths": { "@/*": ["./src/*"] }
  }
}
```

**.env:**
```
EXPO_PUBLIC_API_URL=https://api.example.com
EXPO_PUBLIC_APP_ENV=development
```

### 5. Setup Design Tokens
Create `constants/colors.ts`, `constants/spacing.ts`, `constants/config.ts` following `.agents/rules/naming-conventions.md`.

### 6. Setup Core Services
- `services/api.ts` — Axios instance with interceptors (see `.agents/rules/api-integration.md`)
- `stores/auth.store.ts` — Auth store with SecureStore (see `.agents/rules/state-management.md`)
- `utils/logger.ts` — Structured logger (see `.agents/rules/logging.md`)
- `utils/error.ts` — AppError class (see `.agents/rules/error-handling.md`)
- `i18n/index.ts` — i18next config (see `.agents/rules/i18n.md`)

### 7. Verify Setup
// turbo
```bash
npx expo start
```
- [ ] App runs on iOS simulator
- [ ] App runs on Android emulator
- [ ] Navigation works between screens
- [ ] No TypeScript errors

### 8. Initial Commit
```
feat: initial project setup

- Expo + TypeScript + Expo Router
- Zustand + TanStack Query + Axios
- i18n (vi, en)
- Design tokens, logger, error handling
- Auth flow with SecureStore
```
