---
description: Stage, commit, push code và tạo Pull Request theo Conventional Commits
---

## Steps

### 1. Kiểm tra thay đổi
// turbo
```bash
git status
```

// turbo
```bash
git diff --stat
```

Review danh sách files thay đổi — đảm bảo không commit file không mong muốn.

### 2. Chạy Tests & Lint trước khi commit
// turbo
```bash
npm test -- --watchAll=false
```

// turbo
```bash
npm run lint
```

Nếu tests hoặc lint fail → **STOP** — fix trước khi commit.

### 2.5. Bắt buộc viết Unit Test (Mandatory Test Check)
Trước khi commit, đặc biệt là với các loại `feat` hoặc `fix`, bạn **PHẢI** có Unit Test đi kèm:
- **Tính năng mới (`feat`)**: Bắt buộc viết test cho các tầng xử lý (services, hooks, stores, utils) và UI rendering chính.
- **Sửa lỗi (`fix`)**: Bắt buộc tạo regression test cho file bị lỗi để chặn bug tái diễn trong tương lai.
- 🔴 **DỪNG LẠI (STOP):** Nếu feature hoặc bug fix hiện tại chưa có file test đính kèm, quá trình commit phải bị chặn (block) lại. **AI phải nhắc user chạy lệnh `/write-tests [path]` trước khi thực hiện viết commit message**.

### 3. Stage Files
```bash
# Stage tất cả
git add .

# Hoặc stage từng file/folder
git add src/components/orders/
git add src/services/orders.service.ts
```

> ⚠️ Kiểm tra lại với `git status` sau khi stage — đừng commit files không liên quan.

### 4. Commit (Conventional Commits)

Format: `<type>(<scope>): <description>`

> ⚠️ **Quy định độ dài**: Dòng tiêu đề (subject) của commit message **KHÔNG ĐƯỢC VƯỢT QUÁ 60 KÝ TỰ**. Phải viết thật ngắn gọn, đi thẳng vào vấn đề. Nếu cần giải thích chi tiết, hãy xuống dòng viết vào phần body.

```bash
# Feature mới
git commit -m "feat(orders): add order list screen with filter and pull-to-refresh"

# Bug fix
git commit -m "fix(auth): resolve token refresh race condition on Android"

# Refactor
git commit -m "refactor(orders): extract OrderCard into separate component"

# Test
git commit -m "test(orders): add unit tests for OrderCard and useOrders hook"

# Chore
git commit -m "chore: upgrade expo sdk to v52"

# Nhiều thay đổi → body
git commit -m "feat(orders): add order management feature

- Added OrderList screen with pagination and filter
- Added OrderCard component with accessibility
- Added orders.service with TanStack Query hooks
- Added translations (vi, en)
- Added unit tests (85% coverage)

Closes #123"
```

### Types

| Type | Khi nào dùng |
|------|-------------|
| `feat` | Feature mới |
| `fix` | Bug fix |
| `refactor` | Refactor code (không thay đổi behavior) |
| `test` | Thêm/sửa tests |
| `style` | Format code (không thay đổi logic) |
| `docs` | Cập nhật docs |
| `chore` | Dependencies, config, CI/CD |
| `perf` | Performance improvement |

### 5. Push
```bash
# Push branch hiện tại
git push origin HEAD

# Push branch mới lần đầu
git push -u origin feature/order-list-screen
```

### 6. Tạo Pull Request (nếu cần)

PR title theo format: `[type] Short description`

PR body template:
```markdown
## Mô tả
- Tóm tắt thay đổi

## Screenshots / Videos
- Screenshot trước/sau (nếu UI change)

## Checklist
- [ ] Tests pass
- [ ] Lint pass
- [ ] Code review checklist followed (/mobile-review)
- [ ] Tested on iOS
- [ ] Tested on Android
- [ ] Translations added (vi + en)
- [ ] No hardcoded strings
```

### 7. Sau khi Push
- [ ] PR created / updated
- [ ] CI/CD pipeline passes
- [ ] Assign reviewer
- [ ] Link to Jira/task nếu có
