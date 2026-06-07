# TypeScript Best Practices

## Compiler Configuration

Enable strict mode in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true
  }
}
```

## Type Rules

- **No `any`**. Use `unknown` + type guards for dynamic data:

```typescript
function parseResponse(data: unknown): User {
  if (!data || typeof data !== 'object') throw new Error('Invalid response');
  if (!('id' in data) || !('name' in data)) throw new Error('Missing fields');
  return data as User; // Safe after validation
}
```

- **Explicit Props interfaces** for every component:

```typescript
interface AvatarProps {
  uri: string;
  size?: number;
  onPress?: () => void;
}

export function Avatar({ uri, size = 48, onPress }: AvatarProps) { ... }
```

- **Use `as const`** for constant objects and arrays:

```typescript
const ROUTES = {
  HOME: '/',
  PROFILE: '/profile',
  SETTINGS: '/settings',
} as const; // Readonly, literal types

// Usage: typeof ROUTES.HOME === '/' (literal, not string)
```

- **Discriminated unions** for state machines:

```typescript
type AuthState =
  | { status: 'idle' }
  | { status: 'authenticating' }
  | { status: 'authenticated'; user: User }
  | { status: 'unauthenticated'; error?: string };
```

## Naming Conventions

| Construct | Convention | Example |
|-----------|------------|---------|
| Interface | PascalCase, no `I` prefix | `UserProfile`, `CartItem` |
| Type alias | PascalCase | `ApiResponse<T>`, `ThemeColors` |
| Component | PascalCase | `UserCard`, `PrimaryButton` |
| Hook | camelCase, `use` prefix | `useAuth`, `useFetchPosts` |
| Utility fn | camelCase | `formatDate`, `debounce` |
| Constant | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `API_BASE_URL` |
| Enum | PascalCase | `PaymentStatus`, `UserRole` |
| Generic param | `T`, `K`, `V` or descriptive | `T`, `TData`, `TError` |

## Type Imports

Use `import type` for type-only imports. Reduces runtime imports and clarifies usage:

```typescript
// ✅ Type-only import
import type { User, Post } from '@/types';

// ❌ Value import when only types are used
import { User, Post } from '@/types';
```

## Utility Types

Leverage built-in utility types:

```typescript
type UserPreview = Pick<User, 'id' | 'name' | 'avatar'>;
type UserUpdate = Partial<Omit<User, 'id' | 'createdAt'>>;
type UserCreate = Omit<User, 'id' | 'createdAt'>;
```

## Path Aliases

Configure `@/` path aliases in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

Always use aliases instead of relative path spaghetti (`../../components/Button`).
