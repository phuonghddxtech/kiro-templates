# Accessibility — React Native

> Accessibility standards for iOS (VoiceOver) and Android (TalkBack).

## Checklist

- [ ] All interactive elements have `accessibilityRole` and `accessibilityLabel`
- [ ] Images have `accessibilityLabel` describing content
- [ ] Forms have labels accessible to screen readers
- [ ] Color contrast ≥ 4.5:1 for normal text, ≥ 3:1 for large text
- [ ] Touch targets minimum **44x44 points** (Apple HIG) / **48x48 dp** (Material)
- [ ] Loading/error state changes announced via `accessibilityLiveRegion` (Android)
- [ ] Gestures have accessible alternatives (swipe actions → buttons)
- [ ] Screen reader tested on both iOS (VoiceOver) and Android (TalkBack)

---

## Interactive Elements

```tsx
// ✅ Accessible button with minimum touch target
<Pressable
  accessibilityRole="button"
  accessibilityLabel="Xóa đơn hàng"
  accessibilityHint="Nhấn đúp để xóa đơn hàng này"
  style={{ minWidth: 44, minHeight: 44 }}
  onPress={handleDelete}
>
  <Trash2 size={20} color={COLORS.error} accessibilityElementsHidden />
</Pressable>
```

---

## Images

```tsx
// ✅ Decorative image — hide from screen reader
<Image source={backgroundImage} accessibilityElementsHidden />

// ✅ Meaningful image — describe content
<Image
  source={{ uri: product.image }}
  accessibilityLabel={`Ảnh sản phẩm ${product.name}`}
/>
```

---

## Forms

```tsx
// ✅ Label accessible to screen reader
<Input
  label="Email"
  accessibilityLabel="Email đăng nhập"
  value={email}
  onChangeText={setEmail}
/>
```

---

## State Announcements

```tsx
// ✅ Announce loading state changes (Android)
<View accessibilityLiveRegion="polite">
  {isLoading ? <Text>Đang tải...</Text> : <Text>Đã tải xong</Text>}
</View>
```

---

## Color & Typography
- Body text: minimum **16px** (FONT_SIZE.md)
- Never rely on color alone to convey information
- Line height: minimum **1.5x** for body text
- Visible focus indicators on interactive elements
