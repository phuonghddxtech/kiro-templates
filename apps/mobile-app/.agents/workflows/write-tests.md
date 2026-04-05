---
description: Write unit tests for components, hooks, services, and utilities
---

## Steps

### 1. Identify What to Test
Specify the target:
- **Component** → rendering, user interaction, accessibility
- **Service hook** → API calls, cache, error handling
- **Zustand store** → actions, state transitions
- **Utility function** → input/output, edge cases
- **Custom hook** → behavior, return values

### 2. Analyze the Source Code
Before writing tests:
- Read the source file carefully
- Identify all **branches** (if/else, ternary, switch)
- Identify all **user interactions** (onPress, onChangeText, onSubmit)
- Identify all **states** (loading, error, empty, data)
- Identify all **props** that change behavior
- List edge cases:
  - null/undefined inputs
  - Empty arrays/strings
  - Network errors
  - Permission denied

### 3. Create Test File
```
# Match source file location
src/components/orders/OrderCard.tsx
  → __tests__/unit/components/OrderCard.test.tsx

src/services/orders.service.ts
  → __tests__/integration/services/orders.service.test.tsx

src/stores/auth.store.ts
  → __tests__/unit/stores/auth.store.test.ts

src/utils/format.ts
  → __tests__/unit/utils/format.test.ts
```

### 4. Write Tests by Type

---

#### 4a. Component Test

```tsx
import { render, fireEvent, screen, waitFor } from '@testing-library/react-native';

describe('ComponentName', () => {
  // Setup: default props, mocks
  const defaultProps = { /* ... */ };

  // Group 1: Rendering
  describe('rendering', () => {
    it('should render correctly with required props', () => {});
    it('should render optional elements when props provided', () => {});
    it('should not render optional elements when props omitted', () => {});
  });

  // Group 2: User Interaction
  describe('interaction', () => {
    it('should call onPress when pressed', () => {});
    it('should call onChangeText with new value', () => {});
    it('should disable button when loading', () => {});
  });

  // Group 3: States
  describe('states', () => {
    it('should show loading indicator when isLoading is true', () => {});
    it('should show error message when error exists', () => {});
    it('should show empty state when data is empty', () => {});
  });

  // Group 4: Accessibility
  describe('accessibility', () => {
    it('should have correct accessibility role', () => {});
    it('should have descriptive accessibility label', () => {});
  });
});
```

**Template:**
```tsx
import { render, fireEvent, screen } from '@testing-library/react-native';
import { OrderCard } from '@/components/orders/OrderCard';

const mockOrder: Order = {
  id: '1',
  code: 'ORD-001',
  total: 150000,
  status: 'pending',
  image: 'https://example.com/image.jpg',
};

describe('OrderCard', () => {
  it('should render order code and formatted total', () => {
    render(<OrderCard order={mockOrder} />);
    expect(screen.getByText('ORD-001')).toBeTruthy();
    expect(screen.getByText('150,000 ₫')).toBeTruthy();
  });

  it('should call onPress with order id when pressed', () => {
    const onPress = jest.fn();
    render(<OrderCard order={mockOrder} onPress={onPress} />);
    fireEvent.press(screen.getByRole('button'));
    expect(onPress).toHaveBeenCalledWith('1');
  });

  it('should have accessible label with order code', () => {
    render(<OrderCard order={mockOrder} />);
    expect(screen.getByLabelText(/ORD-001/)).toBeTruthy();
  });

  it('should truncate long order code with numberOfLines', () => {
    const longOrder = { ...mockOrder, code: 'ORD-VERY-LONG-CODE-12345' };
    render(<OrderCard order={longOrder} />);
    // Render without crash → numberOfLines handles overflow
    expect(screen.getByText(longOrder.code)).toBeTruthy();
  });
});
```

---

#### 4b. Service Hook Test

```tsx
import { renderHook, waitFor } from '@testing-library/react-native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useOrders } from '@/services/orders.service';
import { api } from '@/services/api';

jest.mock('@/services/api');

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
};

describe('useOrders', () => {
  afterEach(() => jest.clearAllMocks());

  it('should return orders when API succeeds', async () => {
    (api.get as jest.Mock).mockResolvedValue({
      data: { success: true, data: [{ id: '1', code: 'ORD-001' }], pagination: { page: 1, total: 1 } },
    });

    const { result } = renderHook(() => useOrders(), { wrapper: createWrapper() });
    await waitFor(() => expect(result.current.isSuccess).toBe(true));

    expect(result.current.data?.data).toHaveLength(1);
    expect(result.current.data?.data[0].code).toBe('ORD-001');
  });

  it('should return error when API fails', async () => {
    (api.get as jest.Mock).mockRejectedValue(new Error('Network error'));

    const { result } = renderHook(() => useOrders(), { wrapper: createWrapper() });
    await waitFor(() => expect(result.current.isError).toBe(true));

    expect(result.current.error).toBeDefined();
  });

  it('should not fetch when enabled is false', () => {
    const { result } = renderHook(
      () => useOrders({ enabled: false }),
      { wrapper: createWrapper() },
    );
    expect(result.current.isFetching).toBe(false);
  });
});
```

---

#### 4c. Zustand Store Test

```ts
import { useAuthStore } from '@/stores/auth.store';
import * as SecureStore from 'expo-secure-store';

jest.mock('expo-secure-store');

describe('useAuthStore', () => {
  beforeEach(() => {
    // Reset store between tests
    useAuthStore.setState({ token: null, user: null, isLoading: true });
    jest.clearAllMocks();
  });

  it('should set token and persist to SecureStore', async () => {
    await useAuthStore.getState().setToken('test-token');

    expect(useAuthStore.getState().token).toBe('test-token');
    expect(SecureStore.setItemAsync).toHaveBeenCalledWith('auth_token', 'test-token');
  });

  it('should clear token and user on logout', async () => {
    useAuthStore.setState({ token: 'test', user: { id: '1', name: 'Test' } as User });

    await useAuthStore.getState().logout();

    expect(useAuthStore.getState().token).toBeNull();
    expect(useAuthStore.getState().user).toBeNull();
    expect(SecureStore.deleteItemAsync).toHaveBeenCalledWith('auth_token');
  });

  it('should load token from SecureStore', async () => {
    (SecureStore.getItemAsync as jest.Mock).mockResolvedValue('stored-token');

    await useAuthStore.getState().loadToken();

    expect(useAuthStore.getState().token).toBe('stored-token');
    expect(useAuthStore.getState().isLoading).toBe(false);
  });
});
```

---

#### 4d. Utility Function Test

```ts
import { formatCurrency, formatDate } from '@/utils/format';

describe('formatCurrency', () => {
  it('should format VND correctly', () => {
    expect(formatCurrency(150000)).toBe('150,000 ₫');
  });

  it('should handle zero', () => {
    expect(formatCurrency(0)).toBe('0 ₫');
  });

  it('should handle large numbers', () => {
    expect(formatCurrency(1_000_000_000)).toBe('1,000,000,000 ₫');
  });

  it('should handle negative numbers', () => {
    expect(formatCurrency(-50000)).toBe('-50,000 ₫');
  });
});
```

---

### 5. Test Naming Convention
```
// ✅ Pattern: should [expected behavior] when [condition]
it('should render order code and total', () => {});
it('should call onPress with order id when pressed', () => {});
it('should show error state when API fails', () => {});
it('should disable submit button when form is submitting', () => {});

// ❌ Bad
it('test 1', () => {});
it('works', () => {});
it('OrderCard', () => {});
```

### 6. Mocking Rules

```ts
// ✅ Mock external modules
jest.mock('@/services/api');
jest.mock('expo-secure-store');
jest.mock('expo-router', () => ({ useRouter: () => ({ push: jest.fn() }) }));

// ✅ Mock i18n
jest.mock('react-i18next', () => ({
  useTranslation: () => ({ t: (key: string) => key }),
}));

// ✅ Mock images
jest.mock('expo-image', () => ({
  Image: 'Image',
}));
```

// turbo
### 7. Run Tests
```bash
npm test -- --watchAll=false
```

// turbo
### 8. Check Coverage
```bash
npm test -- --coverage --watchAll=false
```

Coverage must be **≥ 80%** for:
- [ ] Statements
- [ ] Branches
- [ ] Functions
- [ ] Lines

### 9. Checklist
- [ ] All **branches** covered (if/else, ternary)
- [ ] All **user interactions** tested (press, type, submit)
- [ ] All **states** tested (loading, error, empty, data)
- [ ] **Edge cases** covered (null, empty, large data)
- [ ] **Accessibility** verified (role, label)
- [ ] **Mocks** cleaned up (`jest.clearAllMocks()` in beforeEach/afterEach)
- [ ] Test names follow `should [behavior] when [condition]`
- [ ] No `console.log` in tests
- [ ] Coverage ≥ 80%
