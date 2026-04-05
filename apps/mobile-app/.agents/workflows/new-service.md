---
description: Create a new API service with TanStack Query hooks and TypeScript types
---

## Steps

### 1. Define Types
Create types for the API resource:
```
src/types/{feature}.types.ts
```
```tsx
export interface Order {
  id: string;
  code: string;
  total: number;
  status: OrderStatus;
  createdAt: string;
}

export type OrderStatus = 'pending' | 'processing' | 'completed' | 'cancelled';

export interface CreateOrderDto {
  customerName: string;
  phone: string;
  items: OrderItem[];
}
```

### 2. Create Service File
```
src/services/{feature}.service.ts
```

### 3. Implement Query Key Factory
```tsx
const KEYS = {
  all: ['{feature}'] as const,
  list: (params?: Filters) => [...KEYS.all, 'list', params] as const,
  detail: (id: string) => [...KEYS.all, 'detail', id] as const,
};
```

### 4. Implement CRUD Hooks
| Hook | Purpose | HTTP |
|------|---------|------|
| `use{Feature}s(params)` | List with pagination | GET |
| `use{Feature}(id)` | Single item detail | GET |
| `useCreate{Feature}()` | Create new | POST |
| `useUpdate{Feature}()` | Update existing | PUT/PATCH |
| `useDelete{Feature}()` | Delete | DELETE |

Each mutation MUST:
- Invalidate related queries on success
- Log errors with structured logger

### 5. Response Format
Expect consistent API response:
```tsx
// List: { success: true, data: T[], pagination: { page, limit, total, totalPages } }
// Detail: { success: true, data: T }
// Error: { success: false, error: { code, message } }
```

### 6. Write Service Tests
```
__tests__/integration/services/{feature}.service.test.tsx
```
- Mock `api.get`, `api.post`, etc.
- Test successful responses
- Test error handling
- Wrap in `QueryClientProvider`

// turbo
### 7. Run Tests
```bash
npm test -- --watchAll=false
```

### 8. Checklist
- [ ] Types defined for request, response, and entities
- [ ] Query key factory follows consistent pattern
- [ ] `enabled` used to prevent unnecessary requests
- [ ] Mutations invalidate queries on success
- [ ] Errors logged with `logger.error()`
- [ ] No direct API calls from components — only via hooks
