# State Management Patterns — React Native

> When to use Zustand, useState, TanStack Query. Persist and hydration patterns.

## Decision Table

| Data Type | Tool | Example |
|-----------|------|---------|
| **Server data** (API) | TanStack Query | Orders list, user profile from API |
| **Global UI state** | Zustand | Auth token, theme, language, sidebar open |
| **Local UI state** | `useState` | Form input, modal open, accordion expanded |
| **Complex local state** | `useReducer` | Multi-step form, wizard flow |
| **Derived/computed** | `useMemo` | Filtered list, total price |

### ❌ Common Mistakes
```tsx
// ❌ NEVER store API data in Zustand — use TanStack Query
const useStore = create((set) => ({
  orders: [],                    // ❌ Server data in Zustand
  fetchOrders: async () => {     // ❌ Manual fetching
    const data = await api.get('/orders');
    set({ orders: data });
  },
}));

// ✅ Use TanStack Query for server data
function useOrders() {
  return useQuery({ queryKey: ['orders'], queryFn: fetchOrders });
}
```

---

## Zustand Patterns

### Store Structure
```ts
// stores/auth.store.ts
import { create } from 'zustand';

// Separate State and Actions interfaces
interface AuthState {
  token: string | null;
  user: User | null;
  isLoading: boolean;
}

interface AuthActions {
  setToken: (token: string) => Promise<void>;
  logout: () => Promise<void>;
  loadToken: () => Promise<void>;
}

type AuthStore = AuthState & AuthActions;

export const useAuthStore = create<AuthStore>((set, get) => ({
  // State
  token: null,
  user: null,
  isLoading: true,

  // Actions
  setToken: async (token) => {
    await SecureStore.setItemAsync('auth_token', token);
    set({ token });
  },

  logout: async () => {
    await SecureStore.deleteItemAsync('auth_token');
    set({ token: null, user: null });
  },

  loadToken: async () => {
    const token = await SecureStore.getItemAsync('auth_token');
    set({ token, isLoading: false });
  },
}));
```

### Selector Pattern (prevent re-renders)
```tsx
// ✅ Only re-renders when `token` changes
const token = useAuthStore((s) => s.token);
const user = useAuthStore((s) => s.user);

// ❌ Re-renders on ANY state change
const { token, user } = useAuthStore();

// ✅ Multiple selectors with shallow compare
import { shallow } from 'zustand/shallow';

const { token, user } = useAuthStore(
  (s) => ({ token: s.token, user: s.user }),
  shallow,
);
```

### Persist with MMKV
```ts
// stores/app.store.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

const mmkvStorage = {
  getItem: (name: string) => storage.getString(name) ?? null,
  setItem: (name: string, value: string) => storage.set(name, value),
  removeItem: (name: string) => storage.delete(name),
};

export const useAppStore = create<AppStore>()(
  persist(
    (set) => ({
      language: 'vi',
      theme: 'light',
      onboardingDone: false,
      setLanguage: (lang) => set({ language: lang }),
      setTheme: (theme) => set({ theme }),
      completeOnboarding: () => set({ onboardingDone: true }),
    }),
    {
      name: 'app-store',
      storage: createJSONStorage(() => mmkvStorage),
    },
  ),
);
```

### Hydration (wait for persisted state)
```tsx
// app/_layout.tsx
import { useAppStore } from '@/stores/app.store';

export default function RootLayout() {
  const hasHydrated = useAppStore.persist?.hasHydrated() ?? true;

  if (!hasHydrated) return <SplashScreen />;

  return <Slot />;
}
```

---

## TanStack Query Patterns

### Stale/Cache times
```ts
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,       // Data is fresh for 1 min
      gcTime: 5 * 60_000,      // Cache lives for 5 min
      retry: 2,                // Retry failed requests 2 times
      refetchOnWindowFocus: false, // Mobile — no window focus concept
    },
  },
});
```

### Optimistic Updates
```ts
export function useToggleFavorite() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (productId: string) => api.post(`/products/${productId}/favorite`),
    // Optimistic update
    onMutate: async (productId) => {
      await queryClient.cancelQueries({ queryKey: ['products'] });
      const previous = queryClient.getQueryData(['products']);

      queryClient.setQueryData(['products'], (old: Product[]) =>
        old.map((p) => p.id === productId ? { ...p, isFavorite: !p.isFavorite } : p),
      );

      return { previous };
    },
    onError: (_err, _id, context) => {
      queryClient.setQueryData(['products'], context?.previous);
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });
}
```

### Infinite Scroll
```ts
export function useInfiniteOrders() {
  return useInfiniteQuery({
    queryKey: ['orders', 'infinite'],
    queryFn: ({ pageParam = 1 }) =>
      api.get<PaginatedResponse<Order>>('/orders', { params: { page: pageParam } })
        .then((r) => r.data),
    getNextPageParam: (lastPage) =>
      lastPage.pagination.page < lastPage.pagination.totalPages
        ? lastPage.pagination.page + 1
        : undefined,
    initialPageParam: 1,
  });
}
```

---

## Rules

### ✅ MUST
- **TanStack Query** for ALL server/API data
- **Zustand** only for client-side global state (auth, theme, language)
- **useState** for local component state
- **Selectors** for Zustand — never destructure entire store
- **Persist** Zustand stores with MMKV (not AsyncStorage)
- **Tokens** in SecureStore — never in Zustand persist or MMKV

### ❌ NEVER
- Never store API responses in Zustand
- Never fetch data inside Zustand actions — use TanStack Query
- Never destructure full store: `const { ... } = useStore()` — use selectors
- Never persist tokens with MMKV — always SecureStore
