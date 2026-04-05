# AI Agent System cho React Native Development

> Tài liệu tổng hợp cho seminar — Hệ thống Skills, Rules, và Workflows cho Antigravity Agent.

---

## 1. Vấn đề cần giải quyết

### Trước khi có Agent System

| Vấn đề | Hậu quả |
|--------|---------|
| Mỗi dev code theo style riêng | Code review mất thời gian, codebase không đồng nhất |
| Quên handle error/loading/empty states | Bug trên production, UX kém |
| Không follow security best practices | Token lưu sai chỗ, hardcode secrets |
| Mỗi lần code phải tra cứu docs | Chậm, dễ bỏ sót patterns |
| Onboard dev mới mất nhiều thời gian | Phải đọc nhiều docs, hỏi nhiều người |
| Design → Code không pixel-perfect | Dev đoán spacing/color, không khớp Figma |

### Sau khi có Agent System

| Giải pháp | Kết quả |
|-----------|---------|
| **17 Rules** tự động áp dụng | Code luôn đúng chuẩn, không cần nhớ |
| **13 Workflows** step-by-step | Quy trình nhất quán, không bỏ sót bước |
| **Figma MCP** kết nối trực tiếp | Pixel-perfect, map design tokens tự động |
| **Slash commands** | Gõ `/do-task` là AI tự follow toàn bộ quy trình |
| **Self-review checklist** tích hợp | AI tự review code trước khi commit |

---

## 2. Kiến trúc tổng quan

```
.agents/
├── mcp.json                          ← Figma MCP config
├── workflows/                        ← 16 slash commands (3 chung + 13 mobile)
│   ├── deploy.md
│   ├── fix-issue.md
│   ├── review.md
│   ├── analyze-spec.md               ← /analyze-spec
│   ├── do-task.md                    ← /do-task
│   ├── new-screen.md                 ← /new-screen
│   ├── new-component.md              ← /new-component
│   ├── new-service.md                ← /new-service
│   ├── eas-deploy.md                 ← /eas-deploy
│   ├── ota-update.md                 ← /ota-update
│   ├── add-i18n.md                   ← /add-i18n
│   ├── mobile-fix-issue.md           ← /mobile-fix-issue
│   ├── mobile-review.md              ← /mobile-review
│   ├── setup-project.md              ← /setup-project
│   ├── upgrade-sdk.md                ← /upgrade-sdk
│   └── add-native-module.md          ← /add-native-module
│
└── skills/
    └── mobile-developer/
        ├── SKILL.md                  ← Core skill (stack, templates)
        ├── FIGMA_MCP_SETUP.md        ← Hướng dẫn kết nối Figma
        ├── .agents/rules/                    ← 17 rule files
        │   ├── clean-code.md
        │   ├── code-style.md
        │   ├── error-handling.md
        │   ├── security.md
        │   ├── testing.md
        │   ├── performance.md
        │   ├── accessibility.md
        │   ├── api-integration.md
        │   ├── naming-conventions.md
        │   ├── logging.md
        │   ├── i18n.md
        │   ├── figma-integration.md
        │   ├── navigation.md
        │   ├── state-management.md
        │   ├── offline-caching.md
        │   ├── push-notifications.md
        │   └── animation.md
        └── workflows/                ← 13 workflow files
            ├── analyze-spec.md
            ├── do-task.md
            ├── new-screen.md
            ├── new-component.md
            ├── new-service.md
            ├── eas-deploy.md
            ├── ota-update.md
            ├── add-i18n.md
            ├── mobile-fix-issue.md
            ├── mobile-review.md
            ├── setup-project.md
            ├── upgrade-sdk.md
            └── add-native-module.md
```

---

## 3. Rules — Chi tiết

> Rules là các **tiêu chuẩn bắt buộc** mà AI tự động tuân thủ khi viết code.

### Code Quality (4 rules)

| Rule | Nội dung chính |
|------|---------------|
| 🧹 **clean-code** | Variables có nghĩa, functions ≤ 2 args, SOLID, no magic numbers |
| 📐 **code-style** | 2 spaces, PascalCase components, import order, StyleSheet.create() |
| 💥 **error-handling** | AppError class, ErrorBoundary, 4 states (loading/error/empty/data) |
| 📝 **logging** | Structured JSON logger, mandatory fields, NEVER log sensitive data |

### Mobile-Specific (5 rules)

| Rule | Nội dung chính |
|------|---------------|
| ⚡ **performance** | FlatList (không .map), expo-image, useCallback, Reanimated UI thread |
| ♿ **accessibility** | accessibilityRole/Label, touch targets ≥ 44pt, color contrast ≥ 4.5:1 |
| 🧭 **navigation** | Expo Router auth flow, protected routes, deep link validation với Zod |
| 📦 **state-management** | Zustand cho global, TanStack Query cho server, selectors, persist MMKV |
| 🎬 **animation** | Reanimated 3 patterns, skeleton loading, swipe gestures |

### Integration (4 rules)

| Rule | Nội dung chính |
|------|---------------|
| 🌐 **api-integration** | Axios interceptors, TanStack Query hooks, query key factory |
| 🌍 **i18n** | i18next setup, translation files, date/currency format, RTL |
| 📴 **offline-caching** | Network detection, MMKV cache với TTL, offline-first |
| 🔔 **push-notifications** | expo-notifications, permission handling, navigate on tap |

### Security & Standards (4 rules)

| Rule | Nội dung chính |
|------|---------------|
| 🔒 **security** | SecureStore cho tokens, env config, Zod validation, cert pinning |
| 🔑 **naming-conventions** | Cache keys, event names, env vars, design tokens |
| 🧪 **testing** | Jest + RNTL, 80% coverage, regression tests, E2E Detox |
| 🎨 **figma-integration** | Map Figma → RN: colors, spacing, auto-layout, variants |

---

## 4. Workflows — Chi tiết

> Workflows là **quy trình step-by-step** mà AI follow khi thực hiện task.

### Development Flow

```
/analyze-spec → /do-task → /new-service → /new-component → /new-screen → /add-i18n
```

| Workflow | Mục đích | Khi nào dùng |
|----------|---------|------------|
| `/analyze-spec` | Phân tích spec/PRD → chia task breakdown | Nhận feature mới từ PM |
| `/do-task` | Thực hiện 1 task end-to-end | Mỗi task trong breakdown |
| `/new-screen` | Tạo screen mới + routing + 4 states | Cần màn hình mới |
| `/new-component` | Tạo UI component + accessibility + test | Cần component mới |
| `/new-service` | Kết nối API mới + TanStack Query hooks | Cần call API mới |
| `/add-i18n` | Thêm translations hoặc locale mới | Thêm text/ngôn ngữ |

### Maintenance Flow

| Workflow | Mục đích | Khi nào dùng |
|----------|---------|------------|
| `/mobile-fix-issue` | Fix bug trên cả iOS + Android | Có bug report |
| `/mobile-review` | Code review với checklist mobile | Mỗi PR |
| `/upgrade-sdk` | Upgrade Expo SDK an toàn | Mỗi major release |
| `/add-native-module` | Thêm native dependency | Cần camera, maps, etc. |

### Deployment Flow

| Workflow | Mục đích | Khi nào dùng |
|----------|---------|------------|
| `/eas-deploy` | Build + submit App Store / Play Store | Release mới |
| `/ota-update` | OTA fix nhanh (không qua store) | Hotfix JS-only |
| `/setup-project` | Init dự án Expo mới từ đầu | Dự án mới |

---

## 5. Figma MCP Integration

```
Developer:  "/do-task Tạo OrderCard [link Figma]"
    ↓
AI đọc Figma qua MCP
    ↓
Trích xuất:
  • Layout: horizontal, gap 12, padding 16, radius 10
  • Colors: #3B82F6 → COLORS.primary
  • Typography: 16px Semibold → fontSize: FONT_SIZE.md, fontWeight: '600'
  • Components: Image 56x56, 2 Text lines, chevron icon
    ↓
Output: Pixel-perfect React Native component
```

**Không cần**: screenshot, thủ công đo spacing, copy hex colors  
**AI tự động**: đọc Figma → map design tokens → generate code

---

## 6. Kịch bản Demo

### Demo 1: Từ Spec → Code (Full Flow) — 15 phút

> **Mục đích**: Cho thấy AI follow đúng quy trình từ phân tích spec đến code hoàn chỉnh.

**Bước 1** — Phân tích spec (3 phút)
```
/analyze-spec

Feature: Màn hình Danh sách Đơn hàng
- Hiển thị danh sách đơn hàng với ảnh, mã đơn, giá, trạng thái
- Filter theo status: Tất cả, Chờ xử lý, Đang giao, Hoàn thành
- Pull-to-refresh
- Nhấn vào đơn → màn chi tiết
- Empty state khi chưa có đơn

[Paste link Figma nếu có]
```

**Kỳ vọng output**: AI sẽ output bảng screens, components, APIs, task breakdown.

**Bước 2** — Implement task đầu tiên (7 phút)
```
/do-task Tạo API service cho orders (Task 1 trong breakdown)
```

**Highlight cho audience**:
- AI tự tạo types, query key factory, CRUD hooks
- Auto-handle 401 logout
- Structured error logging
- Unit test cho service

**Bước 3** — Tạo screen (5 phút)
```
/do-task Tạo OrderList screen (Task 3 trong breakdown)
```

**Highlight cho audience**:
- 4 states: loading (skeleton) → error → empty → data
- FlatList (không .map), pull-to-refresh
- Accessibility labels
- Translations vi + en
- Self-review checklist tự chạy

---

### Demo 2: Figma → Code (Design to Code) — 10 phút

> **Mục đích**: Cho thấy AI đọc Figma và implement pixel-perfect.

**Bước 1** — Mở Figma file, chọn 1 component (OrderCard)

**Bước 2** — Copy link và gọi workflow
```
/new-component OrderCard https://figma.com/design/xxxxx?node-id=123-456
```

**Highlight cho audience**:
- AI đọc được colors, spacing, typography từ Figma
- Tự map sang `COLORS.*`, `SPACING.*`, `FONT_SIZE.*`
- Tạo component với đúng `StyleSheet.create()`
- Thêm accessibility, translations
- So sánh code output với design → pixel-perfect

---

### Demo 3: Fix Bug Cross-Platform — 5 phút

> **Mục đích**: Cho thấy workflow debug mobile-specific.

**Bước 1** — Report bug
```
/mobile-fix-issue

Bug: Keyboard che input trên Android khi đăng nhập.
iOS hoạt động bình thường.
```

**Highlight cho audience**:
- AI xác định: Android only → `behavior="height"` thay vì `"padding"`
- Tìm root cause, fix, thêm regression test
- Verify cả 2 platforms

---

### Demo 4: Code Review Tự Động — 5 phút

> **Mục đích**: Cho thấy checklist review chi tiết cho mobile.

**Bước 1** — Sau khi code xong, chạy review
```
/mobile-review
```

**Highlight cho audience**:
- AI review 12 categories: code quality, mobile-specific, accessibility, i18n, state, security, API, performance, testing, design matching
- Output report: 🔴 Critical / 🟡 Warning / 🟢 Suggestion / ✅ Good

---

### Demo 5: OTA Hotfix (Bonus — 3 phút)

> **Mục đích**: Cho thấy push fix nhanh không qua App Store.

```
/ota-update

Fix hiển thị sai giá trên OrderCard
```

**Highlight**: Test → push OTA → user tự nhận update khi mở app.

---

## 7. Key Messages cho Seminar

### Thông điệp chính

1. **"Không phải AI thay dev — mà AI enforce best practices mà dev hay quên"**
   - 4 states handling, accessibility, security, error handling
   - Dev vẫn quyết định logic — AI chỉ đảm bảo format đúng

2. **"Rules + Workflows = Institutional Knowledge dạng code"**
   - Thay vì wiki/docs không ai đọc → rules được AI auto-enforce
   - Onboard dev mới: chỉ cần gõ `/do-task` — AI tự follow chuẩn

3. **"Figma MCP = Bridge giữa Design và Code"**
   - Không còn đoán spacing, color
   - Design tokens tự động map → code pixel-perfect

4. **"Invest vào cấu hình ban đầu, tiết kiệm lâu dài"**
   - Setup 1 lần: 17 rules + 13 workflows
   - Benefit mỗi ngày: code nhanh hơn, ít bug hơn, review ít hơn

### Con số

| Metric | Giá trị |
|--------|---------|
| Rules | 17 files — 2,838 lines |
| Workflows | 13 files — 1,121 lines |
| Slash commands | 16 commands |
| Coverage | Clean code → Testing → Deployment → Review |
| Setup time | ~2-3 giờ (1 lần duy nhất) |

---

## 8. Q&A Dự kiến

**Q: Agent có thể chạy mà không cần internet không?**
A: Code generation chạy local. Figma MCP cần internet. OTA/EAS cần internet.

**Q: Nếu rule sai hoặc outdated thì sao?**
A: Sửa file `.md` trong `.agents/rules/` — tất cả sessions sau sẽ dùng rule mới ngay.

**Q: Có áp dụng cho backend/web được không?**
A: Có. Pattern tương tự: tạo skill riêng (đã có `backend-developer`, `frontend-developer`), thêm rules + workflows.

**Q: Chi phí sử dụng?**
A: Phụ thuộc AI model. Rules/workflows miễn phí (chỉ là file `.md`). Figma MCP miễn phí.

**Q: Conflict giữa rules thì sao?**
A: Rules được thiết kế không conflict. Nếu có, rule cụ thể (ví dụ `security.md`) override rule chung (`clean-code.md`).
