# Clean Code — React Native

> Adapted from coding-standards for React Native / TypeScript mobile development.

## 📦 Variables

### ✅ Meaningful, pronounceable names
```tsx
// ✅ Good
const currentUser = useAuthStore((s) => s.user);
const orderItems = data?.items ?? [];

// ❌ Bad — mental mapping
const u = useAuthStore((s) => s.user);
const d = data?.items ?? [];
```

### ✅ No magic numbers — use named constants
```tsx
const DEBOUNCE_MS = 300;
const MIN_PASSWORD_LENGTH = 8;
const PAGE_SIZE = 20;

// ❌ Bad
setTimeout(search, 300);
if (password.length < 8) {}
```

### ✅ Same vocabulary for same type
```tsx
// ❌ getUserInfo(), getClientData(), getCustomerRecord()
// ✅ getUser()
```

### ✅ Use default parameters
```tsx
function fetchOrders(page = 1, limit = PAGE_SIZE) {}
```

### ✅ Don't add redundant context
```tsx
interface Order {
  id: string;      // ✅ not orderId
  total: number;   // ✅ not orderTotal
  status: string;  // ✅ not orderStatus
}
```

---

## 🔧 Functions

### ✅ Max 2 arguments — use object destructuring for more
```tsx
// ❌ function createOrder(name, phone, address, note, items) {}
// ✅
interface CreateOrderParams {
  name: string;
  phone: string;
  address: string;
  note?: string;
  items: OrderItem[];
}
function createOrder({ name, phone, address, note, items }: CreateOrderParams) {}
```

### ✅ Functions should do ONE thing
```tsx
// ❌ function processOrder() { validate(); save(); sendEmail(); }
// ✅ function validateOrder() {}
//    function saveOrder() {}
//    function notifyOrderCreated() {}
```

### ✅ Function names must say what they do
```tsx
// ❌ add(date, 1)
// ✅ addMonthToDate(date, 1)
```

### ✅ No flag parameters — split into separate functions
```tsx
// ❌ function renderList(items, isCompact) {}
// ✅ function renderCompactList(items) {}
//    function renderDetailedList(items) {}
```

### ✅ Avoid side effects — return new data instead of mutating
```tsx
function addItemToCart(cart: CartItem[], item: CartItem): CartItem[] {
  return [...cart, { ...item, addedAt: Date.now() }];
}
```

### ✅ Favor functional programming
```tsx
const totalPrice = orderItems
  .filter((item) => item.quantity > 0)
  .reduce((acc, item) => acc + item.price * item.quantity, 0);
```

### ✅ Encapsulate conditionals
```tsx
// ❌ if (user.role === 'admin' && user.isActive && !user.isBanned) {}
// ✅ if (canAccessAdminPanel(user)) {}
```

### ✅ Avoid negative conditionals
```tsx
// ❌ if (!isNotLoggedIn) {}
// ✅ if (isLoggedIn) {}
```

### ✅ Remove dead code immediately — use git to recover if needed

---

## 🧱 SOLID Principles

| Principle | Rule |
|-----------|------|
| **S** — Single Responsibility | One component/hook = one job |
| **O** — Open/Closed | Open for extension, closed for modification |
| **L** — Liskov Substitution | Subclasses substitutable for base class |
| **I** — Interface Segregation | Don't depend on interfaces you don't use |
| **D** — Dependency Inversion | Depend on abstractions (hooks/interfaces), not concretions |

- **Prefer composition over inheritance** — use hooks and render props

---

## ⚡ Concurrency

### ✅ Always use async/await
```tsx
async function login(email: string, password: string): Promise<User> {
  try {
    const response = await api.post('/auth/login', { email, password });
    return response.data.data;
  } catch (error) {
    throw new AppError('Login failed', getErrorMessage(error));
  }
}

// ❌ Avoid raw promise chains
api.post('/auth/login', { email, password }).then(...).catch(...);
```

---

## 🗒️ Comments

```ts
// ✅ Good — explains WHY, not WHAT
// We debounce 300ms because the search API has rate limiting
const DEBOUNCE_MS = 300;

// ❌ Never leave commented-out code — use git to recover
// ❌ No journal comments (use git log instead)
// ❌ No "obvious" comments: // Set x to 5
```
