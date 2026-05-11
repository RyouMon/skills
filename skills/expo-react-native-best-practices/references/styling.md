# Styling with NativeWind CSS

## Core Rule: NativeWind First

**Always use NativeWind CSS (`className`) for styling.** Use `StyleSheet.create` only for dynamic computed values or animated styles.

## NativeWind Basics

NativeWind brings Tailwind CSS utility classes to React Native via Babel transformation:

```tsx
import { View, Text, TextInput, Pressable } from 'react-native';

export function Button({ onPress, children }) {
  return (
    <Pressable
      onPress={onPress}
      className="flex-row items-center justify-center rounded-lg bg-blue-600 px-4 py-3 active:opacity-80"
    >
      <Text className="text-base font-semibold text-white">{children}</Text>
    </Pressable>
  );
}
```

## Conditional Classes with `cn()`

Use a `cn()` utility (combining `clsx` + `tailwind-merge`) for conditional or computed class names:

```tsx
import { cn } from '@/utils/cn';

// Variant-based styling
<View className={cn(
  'flex-row items-center rounded-lg px-4 py-3',
  variant === 'primary' && 'bg-blue-600',
  variant === 'secondary' && 'bg-gray-200',
  variant === 'danger' && 'bg-red-500',
  isDisabled && 'opacity-50',
)} />

// Responsive sizing
<Text className={cn('text-sm', isTitle && 'text-lg font-bold')} />
```

Example `cn()` implementation:

```typescript
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

## Dark Mode

Configure in `tailwind.config.js`:

```js
module.exports = {
  darkMode: 'class',  // or 'media' for OS-level
  theme: {
    extend: {
      colors: {
        primary: { DEFAULT: '#007AFF', dark: '#0A84FF' },
        surface: { DEFAULT: '#FFFFFF', dark: '#1C1C1E' },
      },
    },
  },
};
```

Use `dark:` prefix in components:

```tsx
<View className="bg-white dark:bg-gray-900">
  <Text className="text-gray-900 dark:text-white">
    Adapts to theme
  </Text>
</View>
```

## Design Tokens

Define all colors, spacing, and typography in `tailwind.config.js`, not inline:

```js
theme: {
  extend: {
    colors: {
      brand: {
        50: '#E6F2FF', 100: '#CCE5FF', 500: '#007AFF', 600: '#005FCC',
      },
    },
    spacing: { 18: '4.5rem', 88: '22rem' },
    fontFamily: { sans: ['Inter', 'system-ui', 'sans-serif'] },
  },
}
```

Then reference in components: `className="bg-brand-500 text-brand-50"`.

## When to Use StyleSheet.create

Limited exceptions where `StyleSheet.create` or inline styles are acceptable:

1. **Dynamic values computed at runtime** (position, size from gesture or layout):

```tsx
const dynamicStyle = useMemo(() => ({
  width: containerWidth * 0.8,
  left: slidePosition,
}), [containerWidth, slidePosition]);
```

2. **Animated styles with Reanimated**:

```tsx
const animatedStyle = useAnimatedStyle(() => ({
  transform: [{ translateY: offset.value }],
  opacity: opacity.value,
}));
```

3. **Third-party libraries that only accept style objects** (rare).

## Anti-Patterns

```tsx
// ❌ Inline style object — new reference every render
<View style={{ flex: 1, justifyContent: 'center' }} />

// ✅ NativeWind className
<View className="flex-1 justify-center" />

// ❌ Mixing NativeWind and StyleSheet for the same property (conflict-prone)
<View className="bg-blue-500" style={{ backgroundColor: 'red' }} />

// ✅ Pick one system per property, or use style only for dynamic override
<View className="bg-blue-500" style={{ opacity: animatedOpacity }} />
```

## Responsive Design

Use Tailwind responsive prefixes for screen-size variants:

```tsx
<View className="w-full md:w-1/2 lg:w-1/3" />
```

For platform-specific differences, prefer file extensions (`Component.ios.tsx`) over runtime `Platform.OS` checks in class names.
