# Code Style — React Native

> Formatting, naming, and import conventions for React Native / TypeScript.

## Formatting
- Indentation: **2 spaces** (no tabs)
- Max line length: **100 characters**
- Use **single quotes** for strings
- Always use **semicolons**
- Trailing commas in multi-line structures

---

## Naming Casing

```tsx
// Variables and functions: camelCase
const userProfile = {};
function getUserById(id: string) {}

// Components, Classes, Interfaces, Types: PascalCase
function OrderCard() {}
class AuthService {}
interface UserProfile {}
type OrderStatus = 'pending' | 'completed';

// Constants: UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const API_TIMEOUT_MS = 15_000;
```

---

## File Naming

```
# Component files: PascalCase (React Native convention)
OrderCard.tsx, EmptyState.tsx, Button.tsx

# Non-component files: kebab-case
auth.store.ts, orders.service.ts, api.types.ts

# Test files: match source file + .test
OrderCard.test.tsx, auth.store.test.ts

# One component per file — tên file = tên component
```

---

## Import Order

```tsx
// 1. React / React Native
import { useState, useCallback } from 'react';
import { View, Text, FlatList } from 'react-native';

// 2. External deps
import { useQuery } from '@tanstack/react-query';
import { useRouter } from 'expo-router';

// 3. Internal modules
import { OrderCard } from '@/components/orders/OrderCard';
import { useOrders } from '@/services/orders.service';
import { COLORS, SPACING } from '@/constants';
```

---

## Component Style Rules

- **One component per file** — name file same as component (PascalCase)
- **Always use `StyleSheet.create()`** — never inline styles
- **Extract constants** — colors, spacing, radii into `constants/`
- **Use `Pressable`** over `TouchableOpacity` (more flexible, better accessibility)
- **Use `expo-image`** for all images — provides caching, transitions, and recycling
- **`numberOfLines`** on all text that could overflow

---

## File Organization
- One class/service per file
- Group related files in feature folders
- Index files for clean imports
