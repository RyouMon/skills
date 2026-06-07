# Architecture & Directory Structure

## Feature-Based Organization

Each feature module contains all code for that business domain:

```
features/auth/
  components/       # Auth-only components
  hooks/            # useAuth, useLogin, useLogout
  services/         # authApi.ts — login(), register(), refreshToken()
  stores/           # authStore.ts (Zustand)
  types.ts          # AuthUser, LoginPayload, AuthState
features/profile/
  components/
  hooks/
  services/
  stores/
  types.ts
```

Rules:
- A feature module does NOT import from sibling features. Shared code goes through `src/components/`, `src/hooks/`, etc.
- Co-locate tests with source files (same directory or `__tests__/` subfolder).

## Expo Router Patterns

### Route Files as Thin Wrappers

Route files delegate to feature components. Keep them minimal:

```tsx
// src/app/(tabs)/profile.tsx
import { ProfileScreen } from '@/features/profile/components/ProfileScreen';

export default function ProfileRoute() {
  return <ProfileScreen />;
}
```

### Route Groups

Use parentheses for URL-less grouping:
- `(tabs)` — bottom tab screens
- `(auth)` — authentication flow
- `(modals)` — modal presentations

### Root `_layout.tsx`

Replaces `App.tsx`. Initialize fonts, splash screen, theme providers here:

```tsx
// src/app/_layout.tsx
import { Stack } from 'expo-router';
import { ThemeProvider } from '@/theme/ThemeProvider';

export default function RootLayout() {
  return (
    <ThemeProvider>
      <Stack screenOptions={{ headerShown: false }} />
    </ThemeProvider>
  );
}
```

### Navigation

- Prefer `<Link href="/path" />` for declarative navigation.
- Use `useRouter()` (`push`, `replace`, `back`) for imperative navigation.
- Read params with `useLocalSearchParams()`.

### Route Guards

Check auth state in the layout. Redirect unauthenticated users:

```tsx
// src/app/(app)/_layout.tsx
import { Redirect, Stack } from 'expo-router';
import { useAuth } from '@/features/auth/hooks/useAuth';

export default function AppLayout() {
  const { isAuthenticated } = useAuth();
  if (!isAuthenticated) return <Redirect href="/login" />;
  return <Stack />;
}
```

## Platform-Specific Code

Use file extensions for platform splits:
- `Component.native.tsx` — iOS + Android
- `Component.ios.tsx` — iOS only
- `Component.android.tsx` — Android only
- `Component.tsx` — Web (or fallback)

Expo's module resolution picks the correct file automatically. Prefer this over runtime `Platform.OS` checks.
