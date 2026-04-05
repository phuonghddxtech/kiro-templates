# Animation Patterns — React Native

> Reanimated 3, Gesture Handler, shared transitions, and skeleton loading.

## Approved Stack

| Tool | Use Case |
|------|----------|
| **Reanimated 3** | Complex animations (runs on UI thread) |
| **Gesture Handler** | Swipe, pinch, pan gestures |
| **LayoutAnimation** | Simple layout changes (add/remove items) |
| **expo-image transition** | Image load animation |

> ❌ Do NOT use old `Animated` API — always use Reanimated 3.

---

## Basic Animated Value

```tsx
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  withTiming,
} from 'react-native-reanimated';

export function AnimatedCard() {
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  const handlePressIn = () => {
    scale.value = withSpring(0.95);
  };

  const handlePressOut = () => {
    scale.value = withSpring(1);
  };

  return (
    <Pressable onPressIn={handlePressIn} onPressOut={handlePressOut}>
      <Animated.View style={[styles.card, animatedStyle]}>
        {/* content */}
      </Animated.View>
    </Pressable>
  );
}
```

---

## Fade In on Mount

```tsx
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';

// ✅ Built-in entering/exiting animations
<Animated.View entering={FadeIn.duration(300)} exiting={FadeOut.duration(200)}>
  <Text>Hello</Text>
</Animated.View>

// Staggered list items
{items.map((item, index) => (
  <Animated.View
    key={item.id}
    entering={FadeIn.delay(index * 50).duration(300)}
  >
    <ItemCard item={item} />
  </Animated.View>
))}
```

---

## Skeleton Loading

```tsx
// components/feedback/Skeleton.tsx
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withRepeat,
  withTiming,
} from 'react-native-reanimated';
import { useEffect } from 'react';

interface SkeletonProps {
  width: number | `${number}%`;
  height: number;
  borderRadius?: number;
}

export const Skeleton: FC<SkeletonProps> = ({ width, height, borderRadius = RADIUS.sm }) => {
  const opacity = useSharedValue(0.3);

  useEffect(() => {
    opacity.value = withRepeat(withTiming(0.7, { duration: 800 }), -1, true);
  }, []);

  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
  }));

  return (
    <Animated.View
      style={[{ width, height, borderRadius, backgroundColor: COLORS.border }, animatedStyle]}
    />
  );
};

// Usage in LoadingState
export function OrderCardSkeleton() {
  return (
    <View style={styles.card}>
      <Skeleton width={56} height={56} borderRadius={RADIUS.sm} />
      <View style={{ flex: 1, gap: 8, marginLeft: SPACING.md }}>
        <Skeleton width="60%" height={16} />
        <Skeleton width="40%" height={14} />
      </View>
    </View>
  );
}
```

---

## Swipe to Delete (Gesture Handler)

```tsx
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  runOnJS,
} from 'react-native-reanimated';

export function SwipeableRow({ children, onDelete }: Props) {
  const translateX = useSharedValue(0);

  const pan = Gesture.Pan()
    .onUpdate((e) => {
      translateX.value = Math.min(0, e.translationX); // Only swipe left
    })
    .onEnd((e) => {
      if (e.translationX < -100) {
        runOnJS(onDelete)();
      }
      translateX.value = withSpring(0);
    });

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
  }));

  return (
    <GestureDetector gesture={pan}>
      <Animated.View style={animatedStyle}>{children}</Animated.View>
    </GestureDetector>
  );
}
```

---

## Rules

### ✅ MUST
- **Reanimated 3** for all animations (UI thread, 60fps)
- **`useSharedValue`** + **`useAnimatedStyle`** pattern
- **Skeleton loading** instead of spinners for content loading
- **`withSpring`** for interactive elements (feels natural)
- **`withTiming`** for UI transitions (predictable duration)

### ❌ NEVER
- Never use old `Animated` API from React Native
- Never run heavy JS on the UI thread — use `runOnJS()` for JS callbacks
- Never animate layout properties (width/height) without `LayoutAnimation`
- Never use `setTimeout` for animations — use Reanimated timing
