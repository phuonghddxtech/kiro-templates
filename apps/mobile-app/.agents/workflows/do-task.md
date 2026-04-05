---
description: Execute a mobile development task end-to-end with planning, implementation, testing, and review
---

## Steps

### 0. Khuyến nghị Model
Trước khi bắt đầu, phân tích task và đưa ra khuyến nghị model phù hợp theo `.agents/rules/model-selection.md`:
- Đánh giá độ phức tạp: Đơn giản / Trung bình / Phức tạp
- Kiểm tra: có cần multimodal (ảnh, video) không? Có liên quan 10+ files không?
- Output khuyến nghị theo format chuẩn:
```
💡 **Khuyến nghị model:** [Model name]
📋 **Loại task:** [Đơn giản / Trung bình / Phức tạp]
💰 **Chi phí ước tính:** [$ / $$ / $$$]
⚡ **Tốc độ:** [Nhanh / Vừa / Chậm]
🔄 **Model hiện tại:** [Model đang dùng]
➡️ **Cần đổi không:** [Có / Không]
```

### 1. Understand the Task
- Read the task description / Jira ticket carefully
- Identify the type of work:
  - 🆕 **New feature** → may need: screen, components, service, translations
  - 🐛 **Bug fix** → follow `/mobile-fix-issue` workflow instead
  - ♻️ **Refactor** → identify scope and impact
  - 📝 **Chore** → dependency update, config change
- List acceptance criteria (ACs) from the task
- Ask clarifying questions if anything is ambiguous — do NOT assume

### 1.5. Extract Figma Design (if Figma link provided)
If the task includes a **Figma link**:
- Read design data via Figma MCP (see `.agents/rules/figma-integration.md`)
- Extract: layout structure, colors, spacing, typography, component hierarchy
- Map Figma values → existing design tokens (`COLORS.*`, `SPACING.*`, `FONT_SIZE.*`)
- If new tokens needed → add to `constants/`
- List all components visible in design that need to be created
- Note any states/variants shown (default, pressed, disabled, empty)

### 2. Plan the Implementation
Create a checklist of what needs to be done:
```markdown
- [ ] Types / interfaces
- [ ] Service hooks (API)
- [ ] Zustand store (if global state needed)
- [ ] Components
- [ ] Screen(s)
- [ ] Translations (vi + en)
- [ ] Unit tests
- [ ] Platform handling (iOS/Android)
```

Review the plan with the user before proceeding.

### 3. Read Relevant Rules
Before coding, review the applicable rules in `.agents/rules/`:
- New screen → `.agents/rules/api-integration.md`, `.agents/rules/error-handling.md`
- New component → `.agents/rules/code-style.md`, `.agents/rules/accessibility.md`
- Form → `.agents/rules/security.md` (validation), `.agents/rules/i18n.md`
- All tasks → `.agents/rules/clean-code.md`, `.agents/rules/naming-conventions.md`

### 4. Implement — Layer by Layer
Follow this order to avoid circular dependencies:

**Step 4a: Types**
```
src/types/{feature}.types.ts
```

**Step 4b: Service hooks (if API needed)**
```
src/services/{feature}.service.ts
```
→ Follow `/new-service` workflow

**Step 4c: Store (if global state needed)**
```
src/stores/{feature}.store.ts
```

**Step 4d: Components**
```
src/components/{feature}/{Component}.tsx
```
→ Follow `/new-component` workflow

**Step 4e: Screen(s)**
```
src/app/{route}.tsx
```
→ Follow `/new-screen` workflow

**Step 4f: Translations**
```
src/i18n/locales/vi.json  → add keys
src/i18n/locales/en.json  → add keys
```
→ Follow `/add-i18n` workflow

### 5. Write Tests
- Unit tests for components
- Service tests for API hooks
- Test all 4 states: loading / error / empty / data

// turbo
```bash
npm test -- --watchAll=false
```

### 6. Self-Review
Run through the `/mobile-review` checklist:
- [ ] No hardcoded strings — all `t()`
- [ ] No inline styles — all `StyleSheet.create()`
- [ ] Accessibility labels on interactive elements
- [ ] 4 states handled on every screen
- [ ] Platform differences handled
- [ ] No magic numbers — design tokens used
- [ ] No `console.log` without `__DEV__` guard
- [ ] Security rules followed (SecureStore, Zod validation)
- [ ] **Figma matching** — if Figma link was provided, verify code matches design (colors, spacing, layout)

// turbo
### 7. Lint Check
```bash
npm run lint
```

### 8. Commit
```
feat({scope}): {short description}

- Implemented {feature}
- Added unit tests
- Added translations (vi, en)

Closes #{task-number}
```

### 9. Summary
Produce a brief summary of what was done:
- Files created/modified
- Screenshots or descriptions of UI (if applicable)
- Any decisions made and why
- Known limitations or follow-up tasks
