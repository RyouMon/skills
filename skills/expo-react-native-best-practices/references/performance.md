# Performance Optimization

## Rendering Optimization

### React.memo

Wrap presentational components that receive stable props:

```tsx
import { memo } from 'react';

export const ListItem = memo(function ListItem({ title, onPress }: ItemProps) {
  return <TouchableOpacity onPress={onPress}><Text>{title}</Text></TouchableOpacity>;
});
```

Do NOT memoize components that always receive new object/array references as props — the memo check will always fail.

### useCallback

Stabilize callbacks passed to memoized children:

```tsx
// ✅ Stabilized — ListItem only re-renders when data changes
const handlePress = useCallback((id: string) => { navigate('/detail', { id }); }, []);

// ❌ New function every render — ListItem re-renders every time
const handlePress = (id: string) => { navigate('/detail', { id }); };
```

### useMemo

Cache expensive computations:

```tsx
const sorted = useMemo(() => [...items].sort(compareFn), [items]);
```

## List Optimization

### FlatList / SectionList

Always use `FlatList` or `SectionList` for scrollable data. Never use `ScrollView` + `.map()` for large datasets.

Critical props:

```tsx
<FlatList
  data={items}
  keyExtractor={(item) => item.id}           // Stable, unique keys
  renderItem={({ item }) => <Item {...item} />}
  getItemLayout={(data, index) => (          // O(1) scroll calculations for fixed heights
    { length: 72, offset: 72 * index, index }
  )}
  initialNumToRender={10}                     // Small initial batch
  maxToRenderPerBatch={10}                    // Controlled batch size
  windowSize={5}                              // Viewport + 2 screens each side
  removeClippedSubviews={true}                // Unmount off-screen items
  updateCellsBatchingPeriod={50}              // ms between render batches
/>
```

- Avoid nesting `FlatList` inside `ScrollView` — use `ListHeaderComponent`/`ListFooterComponent` or switch to `SectionList`.
- Avoid nesting `FlatList` inside another `FlatList` — use `SectionList`.

## Image Optimization

- Use appropriate dimensions. Do not download a 2000px image for a 100px thumbnail.
- Prefer WebP/AVIF over PNG/JPEG.
- Use `expo-image` (replaces `react-native-fast-image`) for caching, lazy loading, and format support:

```tsx
import { Image } from 'expo-image';

<Image
  source="https://cdn.example.com/photo.webp"
  contentFit="cover"
  transition={200}
  cachePolicy="memory-disk"
  style={{ width: 200, height: 200 }}
/>
```

- Use low-quality image placeholders (LQIP) or blurhash for perceived performance.

## Animation

- **Always use `react-native-reanimated` v3+**. Animations run on the UI thread — no JS bridge blocking.
- Use `Animated` API only for simple cases and always set `useNativeDriver: true`.
- Never drive animations with `setState` in `requestAnimationFrame` or `setInterval`.

```tsx
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';

const offset = useSharedValue(0);
const animatedStyle = useAnimatedStyle(() => ({
  transform: [{ translateX: offset.value }],
}));

// On gesture end
offset.value = withSpring(targetValue, { damping: 20, stiffness: 200 });
```

## Startup & Bundle

- **Hermes**: Enabled by default in Expo SDK 52+. Reduces APK/IPA size, improves TTI.
- **New Architecture** (Fabric + TurboModules): Enable when all dependencies support it. Reduces bridge overhead.
- **Code splitting**: Use dynamic `import()` for non-critical screens and heavy libraries.
- **Lazy initialization**: Delay analytics, crash reporting, and A/B testing SDKs until after the app is interactive.
- **InteractionManager.runAfterInteractions()**: Defer non-urgent work until gestures/animations complete.

## Memory Management

- Always clean up in `useEffect`:

```tsx
useEffect(() => {
  const subscription = eventEmitter.addListener('event', handler);
  const timer = setInterval(tick, 1000);
  return () => {
    subscription.remove();  // Unsubscribe
    clearInterval(timer);   // Clear timer
  };
}, []);
```

- Do not hold references to large arrays, image blobs, or decoded data after use.
- Cancel in-flight API requests on component unmount (use AbortController or TanStack Query's automatic cancellation).
