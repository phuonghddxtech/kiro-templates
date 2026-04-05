# Navigation & Deep Linking — React Native

> Expo Router navigation patterns, auth flow, deep linking, and modal navigation.

## Expo Router Structure

```
src/app/
├── _layout.tsx              # Root layout — providers, error boundary
├── (auth)/                  # Auth group (unauthenticated)
│   ├── _layout.tsx
│   ├── login.tsx
│   ├── register.tsx
│   └── forgot-password.tsx
├── (tabs)/                  # Tab group (authenticated)
│   ├── _layout.tsx
│   ├── index.tsx            # Home
│   ├── orders.tsx
│   └── profile.tsx
├── orders/
│   └── [id].tsx             # Dynamic route
├── modals/
│   └── filter.tsx           # Modal screen
└── +not-found.tsx           # 404 screen
```

---

## Auth Flow (Protected Routes)

```tsx
// app/_layout.tsx
import { Redirect, Slot } from 'expo-router';
import { useAuthStore } from '@/stores/auth.store';

export default function RootLayout() {
  const { token, isLoading } = useAuthStore();

  if (isLoading) return <SplashScreen />;

  return (
    <ErrorBoundary>
      <QueryClientProvider client={queryClient}>
        <Slot />
      </QueryClientProvider>
    </ErrorBoundary>
  );
}
```

```tsx
// app/(tabs)/_layout.tsx — redirect if not authenticated
import { Redirect, Tabs } from 'expo-router';
import { useAuthStore } from '@/stores/auth.store';

export default function TabLayout() {
  const token = useAuthStore((s) => s.token);

  // ✅ Redirect to login if not authenticated
  if (!token) return <Redirect href="/(auth)/login" />;

  return (
    <Tabs screenOptions={{ headerShown: false }}>
      {/* tabs */}
    </Tabs>
  );
}
```

```tsx
// app/(auth)/_layout.tsx — redirect if already authenticated
import { Redirect, Stack } from 'expo-router';
import { useAuthStore } from '@/stores/auth.store';

export default function AuthLayout() {
  const token = useAuthStore((s) => s.token);

  // ✅ Already logged in → go to tabs
  if (token) return <Redirect href="/(tabs)" />;

  return <Stack screenOptions={{ headerShown: false }} />;
}
```

---

## Navigation Patterns

### Push to screen
```tsx
import { useRouter } from 'expo-router';

const router = useRouter();

// Navigate forward
router.push('/orders/123');
router.push({ pathname: '/orders/[id]', params: { id: '123' } });

// Replace current screen (no back button)
router.replace('/(tabs)');

// Go back
router.back();
```

### Read route params
```tsx
// app/orders/[id].tsx
import { useLocalSearchParams } from 'expo-router';

export default function OrderDetailScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const { data } = useOrder(id);
  // ...
}
```

### Modal screen
```tsx
// app/_layout.tsx
<Stack>
  <Stack.Screen name="(tabs)" />
  <Stack.Screen
    name="modals/filter"
    options={{ presentation: 'modal', headerShown: true, title: 'Filter' }}
  />
</Stack>

// Open modal from anywhere
router.push('/modals/filter');
```

---

## Deep Linking

### Setup in app.config.ts
```ts
export default {
  expo: {
    scheme: 'myapp',
    // Universal Links (iOS) + App Links (Android)
    plugins: [
      ['expo-router', {
        origin: 'https://myapp.com',
      }],
    ],
  },
};
```

### Deep link mapping
```
myapp://orders/123       → app/orders/[id].tsx
https://myapp.com/orders → app/(tabs)/orders.tsx
```

### Handle deep link params safely
```tsx
// ✅ Always validate deep link params
import { useLocalSearchParams } from 'expo-router';
import { z } from 'zod';

const paramSchema = z.object({
  id: z.string().uuid(),
});

export default function OrderDetailScreen() {
  const rawParams = useLocalSearchParams();
  const result = paramSchema.safeParse(rawParams);

  if (!result.success) {
    return <ErrorState message={t('errors.invalidLink')} />;
  }

  const { id } = result.data;
  // Safe to use
}
```

---

## Navigation Rules

### ✅ MUST
- **Always validate** deep link params with Zod before using
- **Auth guard** via layout redirect — not per-screen checks
- **Use `router.push()`** for forward nav, `router.replace()` for auth redirects
- **Type route params** with `useLocalSearchParams<{ id: string }>()`
- **Handle +not-found.tsx** for invalid routes

### ❌ NEVER
- Never navigate inside `useEffect` without checking mount state
- Never pass sensitive data (tokens, passwords) via route params
- Never use screen-level auth checks — use layout-level redirects
- Never hardcode route paths as strings across files — use constants if reused

### Android Back Button
```tsx
import { BackHandler } from 'react-native';
import { useFocusEffect } from 'expo-router';

useFocusEffect(
  useCallback(() => {
    const onBackPress = () => {
      // Custom back behavior
      Alert.alert('Thoát?', 'Bạn muốn thoát ứng dụng?', [
        { text: 'Hủy' },
        { text: 'Thoát', onPress: () => BackHandler.exitApp() },
      ]);
      return true; // prevent default back
    };
    BackHandler.addEventListener('hardwareBackPress', onBackPress);
    return () => BackHandler.removeEventListener('hardwareBackPress', onBackPress);
  }, []),
);
```
