# Error Handling — React Native

> Error handling patterns for React Native mobile apps.

## Core Principles
- **Never swallow errors silently** — always log or rethrow
- Use a **centralized Error Boundary**
- Return **consistent error responses** from API
- Distinguish **operational errors** (expected) vs **programmer errors** (bugs)

---

## Custom Error Class

```ts
// utils/error.ts
import type { AxiosError } from 'axios';

export class AppError extends Error {
  code: string;
  isOperational: boolean;

  constructor(message: string, code = 'UNKNOWN_ERROR') {
    super(message);
    this.name = 'AppError';
    this.code = code;
    this.isOperational = true;
  }
}
```

---

## API Error Extraction

```ts
// utils/error.ts
interface ApiErrorShape {
  success: false;
  error: {
    code: string;
    message: string;
  };
}

export function getErrorMessage(error: unknown): string {
  if (error instanceof Error) {
    const axiosError = error as AxiosError<ApiErrorShape>;

    // API returned structured error
    if (axiosError.response?.data?.error?.message) {
      return axiosError.response.data.error.message;
    }
    if (axiosError.code === 'ECONNABORTED') return 'Kết nối quá chậm. Thử lại sau.';
    if (!axiosError.response) return 'Không có kết nối mạng.';
  }
  return 'Đã xảy ra lỗi không xác định.';
}
```

---

## Global Error Boundary

```tsx
// components/ErrorBoundary.tsx
import { Component, type ReactNode } from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { Button } from '@/components/ui/Button';
import { logger } from '@/utils/logger';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(): State {
    return { hasError: true };
  }

  componentDidCatch(error: Error) {
    // Structured error logging
    logger.error({
      event: 'error_boundary.caught',
      errorName: error.name,
      errorMessage: error.message,
    });

    // Report to crash reporting service in production
    if (!__DEV__) {
      // Sentry.captureException(error);
    }
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <View style={styles.container}>
          <Text style={styles.title}>Đã xảy ra lỗi</Text>
          <Text style={styles.message}>Vui lòng thử lại.</Text>
          <Button
            label="Thử lại"
            onPress={() => this.setState({ hasError: false })}
          />
        </View>
      );
    }
    return this.props.children;
  }
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center', padding: 24 },
  title: { fontSize: 18, fontWeight: '600', marginBottom: 8 },
  message: { fontSize: 14, color: '#64748B', marginBottom: 16 },
});
```

---

## Screen-level Error Handling

Every screen must handle **4 states**: loading → error → empty → data

```tsx
if (isLoading) return <LoadingState />;
if (error) return <ErrorState message="Không thể tải dữ liệu" onRetry={refetch} />;
if (!data?.items?.length) return <EmptyState title="Chưa có dữ liệu" />;
return <DataView data={data} />;
```

---

## Mutation Error Handling

```tsx
const onSubmit = async (data: FormData) => {
  try {
    await createOrder.mutateAsync(data);
  } catch (error) {
    // ✅ Never swallow — always log
    logger.error({
      event: 'order.create_failed',
      message: getErrorMessage(error),
    });
  }
};
```
