---
description: Create a new UI component (design system primitive or feature-specific)
---

## Steps

### 1. Clarify Component Type
- **UI primitive** → `src/components/ui/` (Button, Input, Card, Badge)
- **Feedback** → `src/components/feedback/` (LoadingState, ErrorState, EmptyState)
- **Feature-specific** → `src/components/{feature}/` (OrderCard, UserAvatar)

### 1.5. Extract Figma Design (if Figma link provided)
If a Figma link is provided:
- Read component structure via Figma MCP
- Extract exact styles: colors, padding, gap, radius, typography
- Identify variants → map to component props
- Identify states (default, pressed, disabled) → implement all
- Follow `.agents/rules/figma-integration.md` for mapping rules

### 2. Create Component File
File name: **PascalCase** matching component name.
```
src/components/ui/Badge.tsx
src/components/orders/OrderCard.tsx
```

### 3. Implement Component
Follow this template:
```tsx
import type { FC } from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { COLORS, SPACING, RADIUS } from '@/constants';

interface {ComponentName}Props {
  // Define all props with TypeScript
}

export const {ComponentName}: FC<{ComponentName}Props> = ({ ...props }) => {
  return (
    <View style={styles.container}>
      {/* Implementation */}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    // Use design tokens, not raw numbers
  },
});
```

### 4. Apply Rules
- [ ] `StyleSheet.create()` — no inline styles
- [ ] Design tokens from `constants/` — no magic numbers
- [ ] `Pressable` instead of `TouchableOpacity` (if interactive)
- [ ] `expo-image` instead of RN `Image` (if images)
- [ ] `numberOfLines` on text that could overflow
- [ ] `accessibilityRole` + `accessibilityLabel` on interactive elements
- [ ] Translate all strings with `t()`

### 5. Write Unit Test
```
__tests__/unit/components/{ComponentName}.test.tsx
```
- Test rendering with required props
- Test user interaction (onPress, onChangeText)
- Test accessibility (getByRole, getByLabelText)

// turbo
### 6. Run Tests
```bash
npm test -- --watchAll=false
```

### 7. Export (if shared)
Add to barrel export if UI primitive:
```tsx
// components/ui/index.ts
export { Badge } from './Badge';
```
