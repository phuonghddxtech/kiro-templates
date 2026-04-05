# Offline & Caching — React Native

> Offline-first patterns, MMKV caching, network detection, and retry queues.

## Network Detection

```ts
// hooks/useNetworkStatus.ts
import NetInfo, { useNetInfo } from '@react-native-community/netinfo';

export function useNetworkStatus() {
  const netInfo = useNetInfo();

  return {
    isConnected: netInfo.isConnected ?? true,
    isInternetReachable: netInfo.isInternetReachable ?? true,
    type: netInfo.type, // wifi, cellular, etc.
  };
}
```

### Offline Banner
```tsx
// components/feedback/OfflineBanner.tsx
export const OfflineBanner: FC = () => {
  const { isConnected } = useNetworkStatus();

  if (isConnected) return null;

  return (
    <View style={styles.banner}>
      <WifiOff size={16} color={COLORS.white} />
      <Text style={styles.text}>{t('common.offline')}</Text>
    </View>
  );
};
```

---

## Caching Strategy

### Layer 1: TanStack Query In-Memory Cache
```ts
// Automatic — data stays in memory during session
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,    // Fresh for 1 min
      gcTime: 5 * 60_000,   // Keep in cache 5 min after unused
    },
  },
});
```

### Layer 2: MMKV Persistent Cache
```ts
// utils/cache.ts
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

export const cache = {
  get: <T>(key: string): T | null => {
    const raw = storage.getString(key);
    if (!raw) return null;
    try {
      const { data, expiry } = JSON.parse(raw);
      if (expiry && Date.now() > expiry) {
        storage.delete(key);
        return null;
      }
      return data as T;
    } catch {
      return null;
    }
  },

  set: <T>(key: string, data: T, ttlMs?: number) => {
    const entry = {
      data,
      expiry: ttlMs ? Date.now() + ttlMs : null,
    };
    storage.set(key, JSON.stringify(entry));
  },

  remove: (key: string) => storage.delete(key),

  clear: () => storage.clearAll(),
};
```

### Layer 3: TanStack Query + Persister (Offline-first)
```ts
// services/api.ts
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister';
import { persistQueryClient } from '@tanstack/react-query-persist-client';
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV({ id: 'query-cache' });

const persister = createSyncStoragePersister({
  storage: {
    getItem: (key) => storage.getString(key) ?? null,
    setItem: (key, value) => storage.set(key, value),
    removeItem: (key) => storage.delete(key),
  },
});

// In _layout.tsx
persistQueryClient({ queryClient, persister, maxAge: 24 * 60 * 60 * 1000 }); // 24h
```

---

## Offline-First Patterns

### Show cached data while refreshing
```tsx
export function useOrders() {
  return useQuery({
    queryKey: ['orders'],
    queryFn: fetchOrders,
    placeholderData: () => cache.get<Order[]>('orders'), // Show cached immediately
    staleTime: 60_000,
  });
}
```

### Queue mutations when offline
```ts
// hooks/useOfflineMutation.ts
import { useMutation, onlineManager } from '@tanstack/react-query';

// TanStack Query automatically pauses mutations when offline
// and retries when back online
onlineManager.setEventListener((setOnline) => {
  const unsubscribe = NetInfo.addEventListener((state) => {
    setOnline(!!state.isConnected);
  });
  return unsubscribe;
});
```

---

## Rules

### ✅ MUST
- **Always show cached data** while loading fresh data — never blank screen
- **OfflineBanner** visible when no connection
- **TanStack Query** handles in-memory caching automatically
- **MMKV** for persistent cache — with TTL expiry
- **Graceful degradation** — app usable with cached data when offline
- **Queue mutations** — don't lose user actions when offline

### ❌ NEVER
- Never show empty screen when cached data is available
- Never silently fail when offline — always inform the user
- Never cache sensitive data (tokens, personal info) in MMKV — use SecureStore
