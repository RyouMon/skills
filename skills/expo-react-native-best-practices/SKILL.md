---
name: expo-react-native-best-practices
description: >
  React Native / Expo project coding standards, architecture patterns, and best practices.
  Use when (1) creating or scaffolding a new React Native / Expo project,
  (2) writing or reviewing code in an Expo project (components, hooks, screens, API calls),
  (3) making architecture decisions (state management, navigation, styling approach),
  (4) debugging performance issues or optimizing a React Native app,
  (5) setting up testing, CI/CD, or project structure for Expo.
  Covers TypeScript strict mode, NativeWind CSS, Expo Router, Zustand/React Query,
  performance optimization, testing strategy, and security.
  Triggers on files: *.tsx, *.ts, *.jsx, *.js in apps using expo or react-native packages.
license: MIT
---

# Expo React Native Best Practices

Coding standards and architecture patterns for React Native / Expo projects using TypeScript and NativeWind CSS.

## Core Architecture Rules

Use **Feature-Based** directory structure:

```
src/
  app/              # Expo Router routes ONLY (thin wrappers)
  features/         # Self-contained modules (components, hooks, services, stores, types)
  components/       # Global shared UI primitives (Button, Input, Card)
  hooks/            # Global shared hooks
  utils/            # Pure utility functions (no side effects)
  services/         # API client, network layer
  theme/            # NativeWind theme config, design tokens
```

Rules:
- `src/app/` holds **only route files**. Business logic lives in `features/`.
- Each feature is self-contained. Cross-feature imports go through top-level shared dirs.
- Route files are **thin wrappers**: delegate to feature components.
- Use Expo Router file-based routing: `_layout.tsx` for layouts, `(group)` for route groups.

## Component Patterns

- **Separate presentational from container components**: Screens (containers) handle data; shared UI primitives (presentational) receive props and render.
- **Favor composition over inheritance**: Extend via props/`children`, not modification.
- **Custom hooks**: One hook per file, single responsibility, named `useXxx`. Return data/state, not side-effect functions.
- **Prefer NativeWind `className`** for styling. Use `StyleSheet.create` only for dynamic values or animated styles. See `references/styling.md`.

## State Management Selection

Choose by scope and update frequency:

| Scope | Solution | Example |
|-------|----------|---------|
| Component-local | `useState`, `useReducer` | Form input, toggle |
| Low-frequency shared | `Context` + `useReducer` | Theme, auth status, locale |
| Global / complex | Zustand | Cart, notifications |
| Server data | TanStack Query | API lists, user profiles |

Do NOT use Context for high-frequency updates (causes cascading re-renders).
Normalize state: store by ID, compute derived values with selectors.

## TypeScript Rules

- Enable `strict` mode. No `any` — use `unknown` + type guards.
- Define Props interfaces for every component. Use `import type` for type-only imports.
- Use discriminated unions for async state:

```typescript
type Async<T> =
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };
```

## Performance Checklist

- [ ] Wrap presentational components with `React.memo`.
- [ ] Stabilize callbacks passed to children with `useCallback`.
- [ ] Cache expensive computations with `useMemo`.
- [ ] Use `FlatList` / `SectionList` for data sets. Never `ScrollView` + `map`.
- [ ] Use `react-native-reanimated` for animations (runs on UI thread).
- [ ] Clean up subscriptions, timers, and animations in `useEffect` return.

## Error Handling

- Implement Error Boundaries at root and feature levels. Provide retry UI.
- Handle API errors centrally via interceptor. Distinguish user-facing messages from technical logs.
- Never log passwords, tokens, or PII.

## Security

- Store secrets in env vars (`EXPO_PUBLIC_*`). Never hardcode API keys.
- Use `expo-secure-store` for sensitive local data (not AsyncStorage).
- Validate deep-link parameters before acting on them.

## Gotchas

- `src/app/` is **exclusively** for routes. Placing non-route files there causes Expo Router to treat them as pages.
- NativeWind CSS classes override `StyleSheet` styles when both are applied — be explicit about which system owns each component's styling.
- `npx expo install <pkg>` (not `npm install`) ensures version compatibility with Expo SDK.
- `FlatList` inside `ScrollView` causes scroll conflicts. Use `SectionList` or `ListHeaderComponent`/`ListFooterComponent` instead.
- `useEffect` without cleanup leaks subscriptions and timers. Always return a cleanup function.
- Context value changes re-render all consumers. Split contexts by concern to isolate updates.
- Hermes engine is default on Expo SDK 52+. Do not disable it.
- `.native.tsx` / `.ios.tsx` / `.android.tsx` extensions auto-resolve by platform — no runtime `Platform.OS` checks needed for file-level splits.

## Reference Files

Load the relevant reference for detailed guidance on a specific topic:

- **`references/architecture.md`** — Feature-Based directory deep dive, Expo Router patterns, route guards, platform-specific files.
- **`references/state-management.md`** — State management patterns, Context optimization, Zustand / Redux Toolkit / Jotai selection guide, React Query caching strategy.
- **`references/performance.md`** — `FlatList` tuning props, image optimization, Hermes + new architecture, animation with Reanimated, memory management.
- **`references/styling.md`** — NativeWind CSS conventions, `cn()` utility, `dark:` mode, design tokens in `tailwind.config.js`, when to use `StyleSheet.create`.
- **`references/testing.md`** — Jest + React Native Testing Library setup, component testing patterns, Detox E2E, mocking strategy.
- **`references/security-and-errors.md`** — Error Boundary patterns, structured logging, API error interceptors, secure storage, input validation.
- **`references/typescript.md`** — Strict config, naming conventions, utility types, `as const`, generics patterns.
- **`references/sources.md`** — Citations and references for all principles in this skill.

## Sources

All principles are derived from authoritative sources. See `references/sources.md` for the full citation list.
