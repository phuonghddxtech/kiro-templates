# Logging — React Native

> Structured logging standards for mobile apps.

## Logger Implementation

```ts
// utils/logger.ts
type LogLevel = 'error' | 'warn' | 'info' | 'debug';

interface LogEntry {
  event: string;
  [key: string]: unknown;
}

const LOG_LEVEL_PRIORITY: Record<LogLevel, number> = {
  error: 0, warn: 1, info: 2, debug: 3,
};

const currentLevel: LogLevel = __DEV__ ? 'debug' : 'info';

function shouldLog(level: LogLevel): boolean {
  return LOG_LEVEL_PRIORITY[level] <= LOG_LEVEL_PRIORITY[currentLevel];
}

function log(level: LogLevel, entry: LogEntry) {
  if (!shouldLog(level)) return;

  const structured = {
    level,
    timestamp: new Date().toISOString(),
    service: 'mobile-app',
    env: __DEV__ ? 'development' : 'production',
    ...entry,
  };

  if (__DEV__) {
    console[level === 'error' ? 'error' : level === 'warn' ? 'warn' : 'log'](
      JSON.stringify(structured, null, 2),
    );
  }
  // In production: send to Sentry, Crashlytics, or remote logging
}

export const logger = {
  error: (entry: LogEntry) => log('error', entry),
  warn: (entry: LogEntry) => log('warn', entry),
  info: (entry: LogEntry) => log('info', entry),
  debug: (entry: LogEntry) => log('debug', entry),
};
```

---

## Usage

```ts
// ✅ Structured log — searchable and parseable
logger.info({
  event: 'order.placed',
  orderId: order.id,
  userId: user.id,
  amount: order.total,
  durationMs: Date.now() - startTime,
});

// ❌ Unstructured log
console.log(`Order ${orderId} placed by user ${userId}`);
```

---

## What NOT to Log

```ts
// ❌ NEVER log sensitive data
logger.info({ token: user.token });           // NEVER
logger.info({ password: user.password });     // NEVER
logger.info({ creditCard: payment.card });    // NEVER
```

---

## Production Guard

```ts
// ✅ No console.log in production
if (__DEV__) {
  console.log('Debug info:', data);
}
```

---

## Mandatory Fields
- `level` — error, warn, info, debug
- `timestamp` — ISO 8601
- `service` — 'mobile-app'
- `event` — what happened (e.g. 'order.placed', 'api.request_failed')
- `durationMs` — for performance tracking (when applicable)
