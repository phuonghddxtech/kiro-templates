# Security — React Native

> ⚠️ CRITICAL — These rules must NEVER be violated.

## 🚨 CRITICAL — Never Violate These
- **NEVER** hardcode secrets, API keys, passwords, or tokens in source code
- **NEVER** log sensitive data (passwords, tokens, PII, credit cards)
- **NEVER** use `eval()` or dynamic code execution with user input
- **ALWAYS** validate and sanitize ALL user inputs (with Zod)
- **ALWAYS** store tokens in **`expo-secure-store`** — NEVER AsyncStorage or MMKV

---

## Environment Config

```ts
// ✅ Always use environment variables
// constants/config.ts
export const API_BASE_URL = process.env.EXPO_PUBLIC_API_URL!;
export const APP_ENV = process.env.EXPO_PUBLIC_APP_ENV || 'development';

// ❌ NEVER hardcode
const API_BASE_URL = 'https://api.myapp.com'; // FORBIDDEN
```

```bash
# Environment variables — UPPER_SNAKE_CASE
EXPO_PUBLIC_API_URL=https://api.myapp.com
EXPO_PUBLIC_APP_ENV=production
EAS_PROJECT_ID=...
SENTRY_DSN=...
```

---

## Token Storage

```ts
// ✅ Secure token storage
import * as SecureStore from 'expo-secure-store';

await SecureStore.setItemAsync('auth_token', token);
const token = await SecureStore.getItemAsync('auth_token');
await SecureStore.deleteItemAsync('auth_token');

// ❌ NEVER use for tokens
import AsyncStorage from '@react-native-async-storage/async-storage';
await AsyncStorage.setItem('token', token); // FORBIDDEN — not encrypted
```

---

## Input Validation

```ts
// ✅ Validate all inputs with Zod before sending to API
import { z } from 'zod';

const loginSchema = z.object({
  email: z.string().email().max(255),
  password: z.string().min(8).max(128),
});
```

---

## Authentication
- **JWT** with short expiry: 15 min access token, 7 day refresh token
- Automatic token refresh via Axios interceptor
- Handle 401 → logout user
- **Biometric auth** (FaceID/TouchID) via `expo-local-authentication` for sensitive actions

---

## Security Checklist

- [ ] Tokens stored in **`expo-secure-store`** — NEVER `AsyncStorage` or `MMKV`
- [ ] API base URL from **env config** (`EXPO_PUBLIC_API_URL`) — never hardcoded
- [ ] **Certificate pinning** for production apps handling sensitive data
- [ ] **No sensitive data** in logs
- [ ] **Validate all user inputs** with Zod before sending to API
- [ ] **Obfuscate** production builds (ProGuard for Android, Hermes bytecode)
- [ ] **Deep link validation** — sanitize and validate incoming deep link params
- [ ] **Biometric auth** for sensitive actions
- [ ] **JWT** stored with short expiry — 15min access, 7d refresh
- [ ] **Rate limiting** awareness — handle 429 responses gracefully
