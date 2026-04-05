# Testing — React Native

> Testing standards for React Native apps using Jest + RNTL.

## Testing Pyramid
```
         [E2E Tests — Detox]    ← Few, slow, critical user flows only
       [Integration Tests]      ← Some, test screen + service interaction
     [Unit Tests — Jest+RNTL]   ← Many, fast, test isolated components/hooks
```

## Requirements
- Unit test coverage: **minimum 80%**
- All new features must have tests
- All bug fixes must have a **regression test**
- Tests run in CI before any merge

---

## Test Naming Convention

```ts
describe('OrderCard', () => {
  // ✅ Pattern: should [expected behavior] when [condition]
  it('should render order code and total', () => {});
  it('should call onPress with order id when pressed', () => {});
  it('should have accessible label', () => {});
});
```

---

## Unit Test (Component)

```tsx
// __tests__/components/OrderCard.test.tsx
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
  it('should render order code and total', () => {
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

  it('should have accessible label', () => {
    render(<OrderCard order={mockOrder} />);
    expect(screen.getByLabelText(/ORD-001/)).toBeTruthy();
  });
});
```

---

## Service Test (API Hook)

```tsx
// __tests__/services/orders.service.test.tsx
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
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useOrders', () => {
  it('should return order list when API responds successfully', async () => {
    (api.get as jest.Mock).mockResolvedValue({
      data: { success: true, data: [{ id: '1' }], pagination: { page: 1, total: 1 } },
    });

    const { result } = renderHook(() => useOrders(), { wrapper: createWrapper() });
    await waitFor(() => expect(result.current.isSuccess).toBe(true));

    expect(result.current.data?.data).toHaveLength(1);
  });
});
```

---

## Test File Organization

```
__tests__/
├── unit/
│   ├── components/
│   │   ├── OrderCard.test.tsx
│   │   └── Button.test.tsx
│   ├── stores/
│   │   └── auth.store.test.ts
│   └── utils/
│       └── format.test.ts
├── integration/
│   └── services/
│       └── orders.service.test.tsx
└── e2e/
    └── auth-flow.test.ts        # Detox
```

---

## Test Commands

```bash
npm test                    # Run all tests
npm run test:unit           # Unit tests only
npm run test:coverage       # Generate coverage report
npm run test:watch          # Watch mode for development
```

---

## What to Test
- **Test all components** that contain business logic or user interaction
- **Test all service hooks** with mocked API
- **Test accessibility** — verify `accessibilityLabel` and `accessibilityRole`
- **Snapshot tests** only for design-system primitives (Button, Input, Card)
- **E2E with Detox** for critical user flows (login, purchase, etc.)
