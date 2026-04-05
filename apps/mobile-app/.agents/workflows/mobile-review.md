---
description: Perform a thorough code review with mobile-specific checklist
---

## Steps

### 1. Identify What to Review
- Specify: files, PR, or feature to review
// turbo
```bash
git diff HEAD~1 --stat
```

### 2. Code Quality
- [ ] Clean code: meaningful names, small functions, no magic numbers
- [ ] TypeScript: no `any`, proper interfaces/types
- [ ] Functions ≤ 2 arguments (object destructuring for more)
- [ ] No dead code or commented-out code
- [ ] Single responsibility per component/hook

### 3. Mobile-Specific Checks
- [ ] `FlatList` used for lists — not `.map()`
- [ ] `StyleSheet.create()` used — no inline styles
- [ ] Design tokens used — no raw numbers for colors/spacing
- [ ] **4 states handled**: loading → error → empty → data
- [ ] **Platform differences** handled (iOS vs Android)
- [ ] `expo-image` used — not RN `Image`
- [ ] `Pressable` used — not `TouchableOpacity`
- [ ] `KeyboardAvoidingView` on form screens with correct `behavior` per platform
- [ ] `numberOfLines` on text that could overflow

### 4. Accessibility
- [ ] `accessibilityRole` on all interactive elements
- [ ] `accessibilityLabel` on buttons, icons, images
- [ ] Touch targets ≥ 44x44 (iOS) / 48x48 (Android)
- [ ] Color contrast ≥ 4.5:1

### 5. Internationalization
- [ ] No hardcoded strings — all using `t()`
- [ ] All locale files have the new keys
- [ ] Accessibility labels translated

### 6. State Management
- [ ] Zustand selectors used (not full store destructuring)
- [ ] API data via TanStack Query — not stored in Zustand
- [ ] Query key factory pattern used
- [ ] `useCallback` on handlers passed to list items

### 7. Security
- [ ] No hardcoded secrets or API URLs
- [ ] Tokens in `expo-secure-store` — not AsyncStorage/MMKV
- [ ] Inputs validated with Zod
- [ ] No sensitive data in logs

### 8. API Integration
- [ ] Service hooks used — no direct API calls from components
- [ ] Consistent response types (`ApiSuccessResponse<T>`)
- [ ] Error handling with `getErrorMessage()`
- [ ] Queries invalidated on mutations

### 9. Performance
- [ ] `useCallback`/`useMemo` where needed (list item handlers)
- [ ] `React.memo()` on expensive list item components
- [ ] No `console.log` without `__DEV__` guard
- [ ] Images use `recyclingKey` and `contentFit`

### 10. Design Matching (if Figma link provided)
- [ ] Colors match Figma design → mapped to `COLORS.*`
- [ ] Spacing/padding match Figma → mapped to `SPACING.*`
- [ ] Typography matches Figma → `FONT_SIZE.*` + fontWeight
- [ ] Layout structure matches Figma hierarchy
- [ ] Component variants/states implemented as in Figma

### 11. Testing
- [ ] Unit tests for new components
- [ ] Service tests for new API hooks
- [ ] Bug fixes have regression tests
- [ ] Coverage ≥ 80%

// turbo
### 11. Run Tests
```bash
npm test -- --watchAll=false
```

### 12. Produce Review Report
Format feedback as:
- 🔴 **Critical** — Must fix before merge
- 🟡 **Warning** — Should fix, potential issue
- 🟢 **Suggestion** — Nice to have improvement
- ✅ **Good** — Highlight what's done well
