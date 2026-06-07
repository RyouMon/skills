# Testing Strategy

## Test Pyramid

| Layer | Tool | Target | Coverage |
|-------|------|--------|----------|
| Unit | Jest | Pure functions, hooks, reducers | 80%+ |
| Component | React Native Testing Library | Component render, interaction | Critical paths |
| Integration | Jest + RNTL | Multi-component flows | Key user flows |
| E2E | Detox / Maestro | Full user journeys | Smoke tests |

## Unit Tests (Jest)

Test pure logic, hooks, and reducers:

```typescript
import { renderHook } from '@testing-library/react-hooks';
import { useCounter } from './useCounter';

test('increments count', () => {
  const { result } = renderHook(() => useCounter());
  act(() => result.current.increment());
  expect(result.current.count).toBe(1);
});
```

## Component Tests (React Native Testing Library)

Test from the user's perspective — query by text, label, or role:

```tsx
import { render, screen, fireEvent } from '@testing-library/react-native';
import { LoginForm } from './LoginForm';

test('submits form with email and password', () => {
  const onSubmit = jest.fn();
  render(<LoginForm onSubmit={onSubmit} />);

  fireEvent.changeText(screen.getByLabelText('Email'), 'user@example.com');
  fireEvent.changeText(screen.getByLabelText('Password'), 'password123');
  fireEvent.press(screen.getByRole('button', { name: 'Sign In' }));

  expect(onSubmit).toHaveBeenCalledWith({
    email: 'user@example.com',
    password: 'password123',
  });
});
```

Principles:
- Query elements by `accessibilityLabel`, text content, or ARIA role — not `testID` unless necessary.
- Test **behavior**, not **implementation**. Do not assert on internal state or prop names.
- Mock all external dependencies: Native modules, API calls, navigation.

## Mocking

Mock Native modules and API layer:

```typescript
// __mocks__/expo-secure-store.ts
export const getItemAsync = jest.fn();
export const setItemAsync = jest.fn();
export const deleteItemAsync = jest.fn();

// In test setup
jest.mock('@/services/api', () => ({
  apiClient: { get: jest.fn(), post: jest.fn() },
}));
```

## E2E Tests (Detox / Maestro)

Use Maestro for declarative, low-maintenance E2E tests:

```yaml
# .maestro/login.yaml
appId: com.example.app
---
- launchApp
- tapOn: "Email"
- inputText: "user@example.com"
- tapOn: "Password"
- inputText: "password123"
- tapOn: "Sign In"
- assertVisible: "Welcome"
```

Use Detox for complex flows requiring JS injection or advanced synchronization.

## Test Organization

- Co-locate tests with source files: `Component.tsx` + `Component.test.tsx` in the same directory.
- Or use `__tests__/` subfolder per feature.
- Name test files `*.test.ts` or `*.spec.ts`.

## Code Review Checklist

- [ ] Are API calls mocked?
- [ ] Are Native modules mocked?
- [ ] Do queries reflect user-visible labels/text?
- [ ] Does the test verify behavior, not implementation?
- [ ] Are async operations wrapped in `waitFor`?
