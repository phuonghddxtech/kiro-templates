---
description: Perform a thorough code review of specified files or a pull request
---

## Steps

### 0. Khuyến nghị Model
Đưa ra khuyến nghị model theo `.agents/skills/mobile-developer/rules/model-selection.md`:
- Code review thường → Claude Sonnet 4.6 (nhanh, phát hiện pattern tốt)
- Security review / audit → Claude Opus 4.6 (Thinking)
- Review codebase lớn (nhiều file) → Gemini 3.1 Pro (context window lớn)

### 1. Identify What to Review
- Specify: file, PR, or feature to review
- Review git diff for context:
// turbo
```bash
git diff HEAD~1
```

### 2. Code Quality Check
- [ ] Code follows style guide (clean, consistent naming)
- [ ] No unnecessary complexity or duplication
- [ ] Functions are small and focused (single responsibility)
- [ ] Variable and function names are descriptive

### 3. Security Check
- [ ] No hardcoded secrets or credentials
- [ ] Input validation is present
- [ ] Authentication/authorization checks in place
- [ ] No SQL injection or eval() with user input

### 4. Error Handling Check
- [ ] Errors are properly caught and handled
- [ ] Meaningful error messages
- [ ] No swallowed exceptions
- [ ] Correct HTTP status codes used

### 5. Testing Check
- [ ] Unit tests cover new logic
- [ ] Edge cases are tested
- [ ] Tests are readable and maintainable
- [ ] Coverage threshold (80%) maintained

### 6. Database Check
- [ ] Queries are optimized (no N+1)
- [ ] Transactions used where appropriate

### 7. API Check
- [ ] Endpoints follow REST conventions
- [ ] Request/response schemas are documented

### 8. Produce Review Report
Format feedback as:
- 🔴 **Critical** — Must fix before merge
- 🟡 **Warning** — Should fix, potential issue
- 🟢 **Suggestion** — Nice to have improvement
- ✅ **Good** — Highlight what's done well
