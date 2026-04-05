---
description: Add new translations or support a new locale
---

## Steps

### 1. Identify Scope
- **Adding new keys** (new feature/screen) → go to Step 2
- **Adding new locale** (e.g. Japanese) → go to Step 5

---

### 2. Add Keys to Default Locale First
Always start with `vi.json` (default):
```json
// src/i18n/locales/vi.json
{
  "{feature}": {
    "title": "Tiêu đề",
    "empty": "Chưa có dữ liệu",
    "create": "Tạo mới",
    "creating": "Đang tạo...",
    "detail": "Chi tiết"
  }
}
```

### 3. Copy to All Other Locales
Add the same keys to ALL locale files with translated values:
```json
// src/i18n/locales/en.json
{
  "{feature}": {
    "title": "Title",
    "empty": "No data yet",
    "create": "Create",
    "creating": "Creating...",
    "detail": "Detail"
  }
}
```
> ⚠️ Every locale file MUST have ALL keys — no missing translations.

### 4. Replace Hardcoded Strings
Find and replace all hardcoded strings with `t()`:
```tsx
// ❌ Before
<Text>Chưa có đơn hàng</Text>

// ✅ After
<Text>{t('orders.empty')}</Text>
```

Don't forget accessibility labels:
```tsx
<Pressable accessibilityLabel={t('orders.create')}>
```

---

### 5. Adding a New Locale (e.g. Japanese)

Create the locale file:
```
src/i18n/locales/ja.json
```

Register in i18n config:
```ts
// src/i18n/index.ts
import ja from './locales/ja.json';

i18n.init({
  resources: {
    vi: { translation: vi },
    en: { translation: en },
    ja: { translation: ja },  // ← add
  },
});
```

Add to language picker options:
```tsx
{ label: '日本語', value: 'ja' }
```

### 6. Test with Each Locale
- Switch language in settings
- Verify all screens display correctly
- Check for text overflow (some languages are longer)
- Verify date/currency formatting adapts

// turbo
### 7. Run Tests
```bash
npm test -- --watchAll=false
```

### 8. Checklist
- [ ] All keys exist in ALL locale files
- [ ] No hardcoded strings remain in components
- [ ] Accessibility labels are translated
- [ ] Date and currency use locale-aware formatters
- [ ] Text doesn't overflow (test with longest translation)
- [ ] RTL layout works (if supporting RTL locales)
