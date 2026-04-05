# API Integration — React Native

> API integration patterns using Axios + TanStack Query.

## Axios Instance

```ts
// services/api.ts
import axios from 'axios';
import { QueryClient } from '@tanstack/react-query';
import { useAuthStore } from '@/stores/auth.store';
import { API_BASE_URL, API_TIMEOUT_MS } from '@/constants/config';
import { logger } from '@/utils/logger';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,      // 1 minute
      gcTime: 5 * 60_000,     // 5 minutes
      retry: 2,
      refetchOnWindowFocus: false, // Mobile: no "window focus"
    },
  },
});

export const api = axios.create({
  baseURL: API_BASE_URL,
  timeout: API_TIMEOUT_MS,
  headers: { 'Content-Type': 'application/json' },
});

// Request interceptor — attach token
api.interceptors.request.use((config) => {
  const token = useAuthStore.getState().token;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor — handle 401 + structured logging
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const status = error.response?.status;

    logger.error({
      event: 'api.request_failed',
      url: error.config?.url,
      method: error.config?.method,
      status,
      message: error.message,
    });

    if (status === 401) {
      await useAuthStore.getState().logout();
    }
    return Promise.reject(error);
  },
);
```

---

## Service Layer (TanStack Query Hooks)

```ts
// services/orders.service.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { api } from './api';
import type { Order, CreateOrderDto, PaginatedResponse } from '@/types';

// ✅ Query key factory pattern
const KEYS = {
  all: ['orders'] as const,
  list: (params?: OrderFilters) => [...KEYS.all, 'list', params] as const,
  detail: (id: string) => [...KEYS.all, 'detail', id] as const,
};

// Fetch list with pagination
export function useOrders(params?: OrderFilters) {
  return useQuery({
    queryKey: KEYS.list(params),
    queryFn: () => api.get<PaginatedResponse<Order>>('/api/v1/orders', { params })
      .then((r) => r.data),
  });
}

// Fetch detail
export function useOrder(id: string) {
  return useQuery({
    queryKey: KEYS.detail(id),
    queryFn: () => api.get<{ success: true; data: Order }>(`/api/v1/orders/${id}`)
      .then((r) => r.data.data),
    enabled: !!id,
  });
}

// Create mutation
export function useCreateOrder() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (dto: CreateOrderDto) =>
      api.post<{ success: true; data: Order }>('/api/v1/orders', dto)
        .then((r) => r.data.data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: KEYS.all });
    },
  });
}
```

---

## API Response Format

```ts
// types/api.types.ts
interface ApiSuccessResponse<T> {
  success: true;
  data: T;
  message?: string;
}

interface ApiErrorResponse {
  success: false;
  error: {
    code: string;      // e.g. 'VALIDATION_ERROR'
    message: string;
  };
}

interface PaginatedResponse<T> {
  success: true;
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

---

## Rules

- **Never call API directly from components** — always go through service hooks
- **Query keys** follow a consistent factory pattern (`KEYS.all`, `KEYS.list`, `KEYS.detail`)
- **Use `enabled`** to prevent unnecessary requests
- **Invalidate queries** on mutations — never manually update cache unless optimistic
- **API base URL** from environment config — **never hardcoded**
- **REST conventions**: plural nouns, kebab-case URL, versioned `/api/v1/`
- **Consistent response format**: `{ success, data, pagination }`
