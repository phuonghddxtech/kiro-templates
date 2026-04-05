---
description: Create a new screen with routing, API integration, and all required states
---

## Steps

### 1. Clarify Requirements
- Screen name and route path
- Data source (API endpoint? local state?)
- Parent layout group: `(tabs)`, `(auth)`, or standalone
- Navigation params (if dynamic route like `[id].tsx`)

### 1.5. Extract Figma Design (if Figma link provided)
If a Figma link is provided:
- Read screen layout via Figma MCP
- Identify: header, list/scroll area, cards, empty state, FAB
- Extract colors, spacing, typography → map to design tokens
- List components to create (e.g., OrderCard, FilterBar)
- Follow `.agents/rules/figma-integration.md` for mapping rules

### 2. Create Route File
Create the route file in `src/app/`:
```
src/app/(tabs)/orders.tsx          # Tab screen
src/app/orders/[id].tsx            # Dynamic route
src/app/(auth)/login.tsx           # Auth screen
```

### 3. Create Service Hook (if API needed)
```
src/services/{feature}.service.ts
```
- Define query key factory: `KEYS.all`, `KEYS.list`, `KEYS.detail`
- Create `useList`, `useDetail`, `useCreate` hooks using TanStack Query
- Define TypeScript types in `src/types/{feature}.types.ts`

### 4. Create Feature Components
```
src/components/{feature}/
├── {Feature}Card.tsx
├── {Feature}List.tsx
└── {Feature}Form.tsx (if needed)
```
- Use `StyleSheet.create()` — never inline styles
- Use design tokens from `constants/`
- Add `accessibilityRole` + `accessibilityLabel`

### 5. Implement Screen with 4 States
Every screen MUST handle:
```tsx
if (isLoading) return <LoadingState />;
if (error) return <ErrorState message={t('errors.loadFailed', { resource: t('{feature}.title') })} onRetry={refetch} />;
// Empty state via ListEmptyComponent
// Data state via FlatList renderItem
```

### 6. Add Translations
Add keys to ALL locale files:
```
src/i18n/locales/vi.json → add "{feature}" section
src/i18n/locales/en.json → add "{feature}" section
```

// turbo
### 7. Run Tests
```bash
npm test -- --watchAll=false
```

### 8. Verify
- [ ] 4 states render correctly (loading/error/empty/data)
- [ ] Pull-to-refresh works
- [ ] Navigation to/from screen works
- [ ] Accessibility labels present on interactive elements
- [ ] No hardcoded strings — all using `t()`
- [ ] Platform tested: iOS + Android
