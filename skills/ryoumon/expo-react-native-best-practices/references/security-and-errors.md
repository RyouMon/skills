# Security & Error Handling

## Error Boundaries

Implement at least two levels:

1. **Root level**: Catches all unhandled errors, prevents app crash, shows generic error UI.
2. **Feature level**: Wraps independent modules (dashboard widgets, settings sections) — local errors don't crash the whole app.

```tsx
// src/components/ErrorBoundary.tsx
import { Component, type ReactNode } from 'react';
import { View, Text, Pressable } from 'react-native';

interface Props { children: ReactNode; fallback?: ReactNode; }
interface State { hasError: boolean; error?: Error; }

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <View className="flex-1 items-center justify-center p-6">
          <Text className="text-lg font-semibold text-red-600">Something went wrong</Text>
          <Pressable onPress={() => this.setState({ hasError: false })}>
            <Text className="mt-4 text-blue-600">Retry</Text>
          </Pressable>
        </View>
      );
    }
    return this.props.children;
  }
}
```

## API Error Handling

Centralize in the API client interceptor:

```typescript
// src/services/api.ts
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      authStore.getState().logout();
      router.replace('/login');
    }
    // Format consistent error shape
    return Promise.reject({
      message: error.response?.data?.message || 'Request failed',
      status: error.response?.status,
    });
  },
);
```

Consume errors with discriminated unions:

```tsx
type QueryResult<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };
```

## Logging

- Use a structured logger (e.g., `expo-logger` or a custom wrapper).
- Levels: `debug` (dev only), `info`, `warn`, `error`.
- Include context: userId, route, timestamp.
- **NEVER log**: passwords, tokens, credit card numbers, personal health data.

```typescript
logger.error('Payment failed', {
  userId: user.id,
  route: '/checkout',
  errorCode: err.code,
  // ❌ Never: cardNumber, cvv, token
});
```

## Security Rules

### Secrets & Keys

- Use `EXPO_PUBLIC_*` prefixed environment variables for build-time config.
- Use a secrets manager (Doppler, 1Password Secrets) for sensitive values — never commit `.env` files.
- Validate env vars at startup; fail fast if required vars are missing.

### Local Storage

| Data Type | Storage |
|-----------|---------|
| JWT tokens, API keys | `expo-secure-store` |
| User preferences, cache | `AsyncStorage` |
| Sensitive documents | Keychain (iOS) / Keystore (Android) via `expo-secure-store` |

```typescript
import * as SecureStore from 'expo-secure-store';

await SecureStore.setItemAsync('auth_token', token, {
  keychainService: 'com.example.auth',
});
```

### Network

- Always use HTTPS.
- Implement certificate pinning for high-security apps (banking, healthcare).
- Validate all user input on client AND server.

### Deep Links

Validate parameters before acting:

```typescript
// ❌ Direct navigation without validation
router.push(params.redirectTo);

// ✅ Whitelist validation
const allowedPaths = ['/home', '/profile', '/settings'];
const path = allowedPaths.includes(params.redirectTo)
  ? params.redirectTo
  : '/home';
router.push(path);
```

### Authentication

- Store refresh tokens securely. Access tokens in memory (Zustand store), not storage.
- Implement token refresh with concurrency control (deduplicate simultaneous refresh calls).
- Use short-lived access tokens (15 min) with long-lived refresh tokens (7-30 days).
- Clear all auth state on logout (SecureStore + memory + QueryClient cache).
