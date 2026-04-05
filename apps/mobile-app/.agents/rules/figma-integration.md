# Figma Integration — React Native

> Hướng dẫn AI đọc Figma design qua MCP và map sang React Native code.

## Kết nối Figma MCP

Kiro kết nối Figma qua Remote MCP Server (`https://mcp.figma.com/mcp`).
Config nằm tại `.agents/mcp.json`.

Khi user cung cấp **Figma link**, AI sẽ:
1. Dùng MCP tools để đọc design data
2. Trích xuất styles, layout, components
3. Map sang React Native code pixel-perfect

---

## Workflow khi có Figma link

### Step 1: Đọc design context
Dùng Figma MCP tools để lấy:
- **Layout structure** — hierarchy of frames/components
- **Auto-layout** → map sang Flexbox (`flexDirection`, `gap`, `padding`)
- **Colors** → map sang `COLORS.*` constants
- **Typography** → map sang `FONT_SIZE.*` + `fontWeight`
- **Spacing** → map sang `SPACING.*` constants
- **Corner radius** → map sang `RADIUS.*` constants
- **Shadows** → map sang `Platform.select()` (iOS shadow / Android elevation)
- **Images** → download và dùng với `expo-image`

### Step 2: Map Figma → Design Tokens

```
Figma                          →  React Native
─────────────────────────────────────────────────
Color: #3B82F6                 →  COLORS.primary
Color: #0F172A                 →  COLORS.text
Color: #64748B                 →  COLORS.textSecondary
Spacing: 16px                  →  SPACING.md
Spacing: 8px                   →  SPACING.sm
Spacing: 24px                  →  SPACING.lg
Radius: 10px                   →  RADIUS.md
Radius: 9999px                 →  RADIUS.full
Font: 16px Semibold            →  { fontSize: FONT_SIZE.md, fontWeight: '600' }
Font: 14px Regular             →  { fontSize: FONT_SIZE.sm, fontWeight: '400' }
Shadow: 0 2 8 rgba(0,0,0,0.1) →  Platform.select({ ios: shadow, android: elevation: 4 })
```

### Step 3: Map Figma Auto-Layout → Flexbox

```
Figma Auto-Layout              →  React Native StyleSheet
─────────────────────────────────────────────────
Direction: Horizontal          →  flexDirection: 'row'
Direction: Vertical            →  flexDirection: 'column'
Gap: 12                        →  gap: 12
Padding: 16                    →  padding: SPACING.md
Align: Center                  →  alignItems: 'center'
Justify: Space Between         →  justifyContent: 'space-between'
Fill Container                 →  flex: 1
Hug Contents                   →  (no flex, auto-sized)
Fixed: 56x56                   →  width: 56, height: 56
```

### Step 4: Map Figma Components → RN Components

```
Figma Component                →  React Native
─────────────────────────────────────────────────
Frame                          →  <View>
Text                           →  <Text>
Image/Rectangle with fill      →  <Image> (expo-image)
Button (instance)              →  <Pressable> + <Text>
Input field                    →  <TextInput> or custom <Input>
Icon (instance)                →  Lucide icon component
List of items                  →  <FlatList>
Scroll area                    →  <ScrollView>
Card frame                     →  <View> with shadow + radius
```

### Step 5: Handle Figma Variants → Component Props

```tsx
// Figma: Button variant = { size: 'sm' | 'md' | 'lg', type: 'primary' | 'outline' }
// →

interface ButtonProps {
  size?: 'sm' | 'md' | 'lg';
  variant?: 'primary' | 'outline';
  label: string;
  onPress: () => void;
}
```

---

## Rules khi implement từ Figma

### ✅ MUST
- **Map colors sang `COLORS.*`** — nếu color chưa tồn tại, thêm vào `constants/colors.ts`
- **Map spacing sang `SPACING.*`** — dùng giá trị gần nhất (12px → `SPACING.sm` hoặc tạo mới)
- **Map typography** sang `FONT_SIZE.*` + fontWeight — không dùng raw numbers
- **Giữ đúng hierarchy** từ Figma — nếu Figma có 3 nesting levels, code cũng phải tương tự
- **Export images** từ Figma nếu có — dùng `expo-image` để hiển thị

### ❌ NEVER
- Không bỏ qua spacing/padding từ design — phải khớp chính xác
- Không dùng raw color hex trong code — luôn map sang constants
- Không đoán layout — đọc auto-layout properties từ Figma  
- Không bỏ qua states (pressed, disabled) nếu Figma có define

### Khi Figma value không khớp design tokens

```ts
// Figma dùng spacing 12px nhưng SPACING không có 12
// → Thêm vào constants
export const SPACING = {
  xs: 4,
  sm: 8,
  md: 16,   // existing
  // Thêm giá trị mới nếu cần
} as const;

// Hoặc dùng giá trị gần nhất + note
// padding: SPACING.sm (8px — Figma: 12px, cần thêm SPACING.smd = 12)
```

---

## Ví dụ: Figma → React Native Code

### Figma design:
```
Frame "OrderCard" (auto-layout: horizontal, gap: 12, padding: 16, radius: 10)
├── Image (56x56, radius: 6)
├── Frame "content" (auto-layout: vertical, gap: 2, fill)
│   ├── Text "ORD-001" (16px Semibold, #0F172A)
│   └── Text "150,000 ₫" (14px Regular, #64748B)
└── Icon "chevron-right" (20x20, #94A3B8)
```

### Generated React Native:
```tsx
export const OrderCard: FC<OrderCardProps> = ({ order, onPress }) => (
  <Pressable style={styles.card} onPress={() => onPress?.(order.id)}
    accessibilityRole="button"
    accessibilityLabel={`${t('orders.order')} ${order.code}`}>
    <Image source={{ uri: order.image }} style={styles.image}
      contentFit="cover" recyclingKey={order.id} />
    <View style={styles.content}>
      <Text style={styles.title} numberOfLines={1}>{order.code}</Text>
      <Text style={styles.subtitle}>{formatCurrency(order.total)}</Text>
    </View>
    <ChevronRight size={20} color={COLORS.textMuted} />
  </Pressable>
);

const styles = StyleSheet.create({
  card: {
    flexDirection: 'row',   // ← Figma: horizontal
    gap: 12,                // ← Figma: gap 12
    padding: SPACING.md,    // ← Figma: padding 16
    borderRadius: RADIUS.md, // ← Figma: radius 10
    backgroundColor: COLORS.surface,
    alignItems: 'center',
  },
  image: {
    width: 56, height: 56,  // ← Figma: 56x56
    borderRadius: RADIUS.sm, // ← Figma: radius 6
  },
  content: {
    flex: 1,                // ← Figma: fill
    gap: 2,                 // ← Figma: gap 2
  },
  title: {
    fontSize: FONT_SIZE.md, // ← Figma: 16px
    fontWeight: '600',      // ← Figma: Semibold
    color: COLORS.text,     // ← Figma: #0F172A
  },
  subtitle: {
    fontSize: FONT_SIZE.sm, // ← Figma: 14px
    fontWeight: '400',      // ← Figma: Regular
    color: COLORS.textSecondary, // ← Figma: #64748B
  },
});
```
