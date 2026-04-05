# Naming Conventions — React Native

> Naming standards for cache keys, events, env vars, and project files.

## Cache Keys (Local Storage)

```ts
// Pattern: {app}:{version}:{entity}:{identifier}:{variant}
const CACHE_KEYS = {
  userProfile: (userId: string) => `myapp:v1:user:${userId}:profile`,
  settings: 'myapp:v1:settings',
  onboardingDone: 'myapp:v1:onboarding:completed',
} as const;
```

---

## Event Naming (Analytics/Tracking)

```ts
// Pattern: {entity}.{past_tense_verb}
const EVENTS = {
  USER_LOGGED_IN: 'user.logged_in',
  USER_LOGGED_OUT: 'user.logged_out',
  ORDER_PLACED: 'order.placed',
  ORDER_CANCELLED: 'order.cancelled',
  SCREEN_VIEWED: 'screen.viewed',
} as const;
```

---

## Environment Variables

```bash
# UPPER_SNAKE_CASE
EXPO_PUBLIC_API_URL=https://api.myapp.com
EXPO_PUBLIC_APP_ENV=production
EAS_PROJECT_ID=...
SENTRY_DSN=...
```

---

## File & Folder Naming

```
# Component files: PascalCase
OrderCard.tsx, Button.tsx, EmptyState.tsx

# Non-component files: kebab-case or dot-separated
auth.store.ts, orders.service.ts, api.types.ts

# Folders: lowercase
components/, hooks/, services/, stores/, utils/, constants/, types/

# Test files: match source + .test
OrderCard.test.tsx, auth.store.test.ts
```

---

## Design System Tokens

```ts
// constants/colors.ts — use descriptive semantic names
export const COLORS = {
  primary: '#3B82F6',
  success: '#22C55E',
  error: '#EF4444',
  text: '#0F172A',
  textSecondary: '#64748B',
  background: '#FFFFFF',
  surface: '#F8FAFC',
  border: '#E2E8F0',
} as const;

// constants/spacing.ts — consistent scale
export const SPACING = {
  xs: 4, sm: 8, md: 16, lg: 24, xl: 32, xxl: 48,
} as const;

// constants/radius.ts
export const RADIUS = {
  sm: 6, md: 10, lg: 16, full: 9999,
} as const;
```

---

## Rules
- **Never use raw numbers** for colors, spacing, or sizes — always use constants
- **Consistent token naming** across the app
- **UPPER_SNAKE_CASE** for environment variables
- **PascalCase** for components, **camelCase** for variables/functions
