# State Management

## Decision Flow

1. **Does only one component need it?** → `useState` / `useReducer`
2. **Do a few components share it with infrequent updates?** → `Context`
3. **Is it global app state with complex logic?** → Zustand
4. **Does it come from a server?** → TanStack Query

## Local State

Use `useState` for primitives, `useReducer` for complex objects with multiple interrelated fields.

```tsx
// useReducer for complex form state
const [form, dispatch] = useReducer(formReducer, initialState);
```

## Context Patterns

Split contexts by concern to prevent unnecessary re-renders:

```tsx
// ❌ One context for everything — all consumers re-render on any change
<AppContext.Provider value={{ theme, user, locale }}>

// ✅ Split by concern — each consumer only re-renders when its slice changes
<ThemeContext.Provider><UserContext.Provider><LocaleContext.Provider>
```

- Use `Context` for theme, auth status, locale — data that changes rarely.
- NEVER use Context for high-frequency data (scroll position, animation values, real-time counters).

## Zustand

Default choice for global state. Lightweight, no boilerplate:

```typescript
import { create } from 'zustand';

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
}

export const useCartStore = create<CartStore>((set) => ({
  items: [],
  addItem: (item) => set((s) => ({ items: [...s.items, item] })),
  removeItem: (id) => set((s) => ({ items: s.items.filter((i) => i.id !== id) })),
}));
```

Access with selectors to prevent unnecessary re-renders:

```tsx
// ✅ Selector — only re-renders when items.length changes
const count = useCartStore((s) => s.items.length);

// ❌ No selector — re-renders on any store change
const { items } = useCartStore();
```

## Redux Toolkit

Use for large apps requiring strict data flow, time-travel debugging, or middleware chains:

```typescript
import { createSlice } from '@reduxjs/toolkit';

const authSlice = createSlice({
  name: 'auth',
  initialState: { user: null, token: null },
  reducers: {
    login: (state, action) => { state.user = action.payload.user; state.token = action.payload.token; },
    logout: (state) => { state.user = null; state.token = null; },
  },
});
```

## TanStack Query (React Query)

Default for all server state. Handles caching, deduping, background refetch, pagination:

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Query — cached, auto-refreshed
const { data, isLoading } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
});

// Mutation — with automatic cache invalidation
const queryClient = useQueryClient();
const mutation = useMutation({
  mutationFn: updateUser,
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ['user'] }),
});
```

Keep query keys consistent and hierarchical: `['users']`, `['users', userId]`, `['users', userId, 'posts']`.

## State Normalization

Store collections as records keyed by ID. Do not nest entities.

```typescript
// ✅ Normalized
interface UsersState {
  byId: Record<string, User>;
  allIds: string[];
}

// ❌ Nested — hard to update, causes unnecessary re-renders
interface UsersState {
  users: User[]; // each user contains nested posts, comments...
}
```

Compute derived data with selectors, not in state.
