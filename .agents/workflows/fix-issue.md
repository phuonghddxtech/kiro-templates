---
description: Analyze and fix a reported bug or issue systematically
---

## Steps

### 0. Khuyến nghị Model
Phân tích mức độ nghiêm trọng của issue và khuyến nghị model theo `rules/model-selection.md`:
- Bug thường (UI, logic đơn giản) → Claude Sonnet 4.6
- Bug phức tạp (race condition, memory leak, production crash) → Claude Opus 4.6 (Thinking)
- Cần phân tích screenshot/video bug → Gemini 3.1 Pro
- Output khuyến nghị theo format chuẩn trong `rules/model-selection.md`

### 1. Understand the Issue
- Read the error message or bug description carefully
- Identify the affected component(s)
- Reproduce the issue locally if possible

### 2. Root Cause Analysis
// turbo
```bash
git log --oneline -20
```
- Review affected files
- Look for related tests that may reveal expected behavior

### 3. Plan the Fix
- Identify the minimal change needed
- Consider side effects on other components
- Plan to update or add tests to cover the fix

### 4. Implement
- Make the targeted fix
- Follow clean code principles (see `.agents/skills/clean-code/SKILL.md`)
- Handle errors properly with specific error classes and meaningful messages

### 5. Verify
// turbo
```bash
npm test
npm run lint
```

### 6. Commit
Use Conventional Commit format:
```
fix(scope): short description of the fix

Closes #[issue-number]
```
