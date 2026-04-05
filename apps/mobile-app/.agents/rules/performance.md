# Performance — React Native

> Performance best practices for React Native apps.

## Checklist

- [ ] Lists use `FlatList` with `keyExtractor` — never `.map()` to render arrays
- [ ] Images use `expo-image` with `recyclingKey`, `contentFit`, and `transition`
- [ ] `useCallback` on functions passed to list items to prevent re-renders
- [ ] `useMemo` only where profiled as bottleneck — not by default
- [ ] Heavy operations off the main thread (`InteractionManager` or `requestAnimationFrame`)
- [ ] Animations use `Reanimated` (runs on UI thread) — not `Animated` API
- [ ] No console.log in production — use `__DEV__` guard or structured logger
- [ ] Bundle size monitored — avoid importing entire libraries (tree-shake)
- [ ] `React.memo()` on expensive list item components
- [ ] Splash screen stays visible until app is ready (`expo-splash-screen`)

---

## Lists

```tsx
// ✅ Always use FlatList
<FlatList
  data={data}
  keyExtractor={(item) => item.id}
  renderItem={({ item }) => <OrderCard order={item} onPress={handlePress} />}
  ListEmptyComponent={<EmptyState />}
  refreshControl={<RefreshControl refreshing={isRefetching} onRefresh={refetch} />}
  showsVerticalScrollIndicator={false}
/>

// ❌ Never .map() for lists
{items.map((item) => <OrderCard key={item.id} order={item} />)}
```

---

## Images

```tsx
// ✅ Use expo-image with recycling
import { Image } from 'expo-image';

<Image
  source={{ uri: order.image }}
  style={styles.image}
  contentFit="cover"
  transition={200}
  recyclingKey={order.id}
/>

// ❌ Don't use RN Image (no caching)
import { Image } from 'react-native';
```

---

## Re-render Prevention

```tsx
// ✅ useCallback for handlers passed as props
const handlePress = useCallback((id: string) => {
  router.push(`/orders/${id}`);
}, [router]);

// ✅ Zustand selectors — only re-render on specific state change
const token = useAuthStore((s) => s.token);  // ✅
const { token } = useAuthStore();            // ❌ re-renders on ANY change
```

---

## Production Logging Guard

```ts
// ✅ No console.log in production
if (__DEV__) {
  console.log('Debug info:', data);
}
```
