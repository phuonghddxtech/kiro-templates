---
description: Analyze product spec/PRD and break down into implementable tasks
---

## Steps

### 1. Đọc và Hiểu Spec
- Đọc toàn bộ spec/PRD/user story
- Xác định **mục tiêu chính** của feature
- Identify stakeholders và user personas
- Liệt kê acceptance criteria (ACs)

### 2. Extract Figma Design (nếu có)
Nếu spec đính kèm Figma link:
- Đọc design qua Figma MCP
- List tất cả screens trong flow
- List tất cả components cần tạo
- Trích xuất design tokens mới (nếu có)

### 3. Phân tích Kỹ thuật

#### 3a. Screens
Liệt kê tất cả screens cần tạo/sửa:
```markdown
| Screen | Route | Type | Complexity |
|--------|-------|------|------------|
| OrderList | /(tabs)/orders | Tab screen | Medium |
| OrderDetail | /orders/[id] | Stack screen | High |
| CreateOrder | /orders/create | Modal | High |
```

#### 3b. Components
Liệt kê components cần tạo:
```markdown
| Component | Type | Reusable? | Notes |
|-----------|------|-----------|-------|
| OrderCard | Feature | No | Dùng trong OrderList |
| StatusBadge | UI | Yes | Dùng ở nhiều nơi |
| PriceTag | UI | Yes | Format VND |
```

#### 3c. API Endpoints
Liệt kê APIs cần kết nối:
```markdown
| Endpoint | Method | Hook | Notes |
|----------|--------|------|-------|
| /api/v1/orders | GET | useOrders() | Pagination, filter |
| /api/v1/orders/:id | GET | useOrder(id) | Detail |
| /api/v1/orders | POST | useCreateOrder() | Validate form |
```

#### 3d. State Management
Xác định state cần thiết:
```markdown
| State | Tool | Reason |
|-------|------|--------|
| Order list | TanStack Query | Server data |
| Filter values | useState | Local UI |
| Cart | Zustand + MMKV | Persist across sessions |
```

#### 3e. New Dependencies
Kiểm tra cần thêm package nào:
```markdown
| Package | Purpose | Need dev build? |
|---------|---------|-----------------|
| — | — | — |
```

### 4. Ước tính Complexity

| Tiêu chí | Điểm |
|-----------|------|
| Số screens | + 1 điểm / screen |
| Có form phức tạp | + 2 |
| Cần animation | + 1 |
| Cần push notification | + 2 |
| Cần offline support | + 3 |
| Cần native module mới | + 3 |
| Multi-language | + 1 |
| Có Figma (pixel-perfect) | + 1 |

```
1-3 điểm: Đơn giản (1-2 ngày)
4-7 điểm: Trung bình (3-5 ngày)
8+  điểm: Phức tạp (1-2 tuần)
```

### 5. Chia Task Breakdown

Chia spec thành các tasks nhỏ, mỗi task có thể `/do-task` độc lập:

```markdown
## Task Breakdown

### Phase 1: Foundation
- [ ] Task 1: Tạo types + API service (useOrders, useOrder, useCreateOrder)
- [ ] Task 2: Tạo shared components (StatusBadge, PriceTag)

### Phase 2: Screens
- [ ] Task 3: Tạo OrderList screen với filter + pull-to-refresh
- [ ] Task 4: Tạo OrderDetail screen
- [ ] Task 5: Tạo CreateOrder screen với form validation

### Phase 3: Polish
- [ ] Task 6: Thêm translations (vi + en)
- [ ] Task 7: Thêm skeleton loading + animations
- [ ] Task 8: Unit tests cho tất cả components + services

### Phase 4: Review
- [ ] Task 9: Self-review theo /mobile-review
- [ ] Task 10: Test trên cả iOS + Android
```

### 6. Identify Risks & Dependencies

```markdown
## Risks
| Risk | Impact | Mitigation |
|------|--------|------------|
| API chưa ready | Block Phase 2 | Mock API response |
| Design chưa final | Rework | Confirm with designer trước |

## Dependencies
- [ ] Backend API cho orders phải deploy trước
- [ ] Design Figma phải finalize trước Phase 2
```

### 7. Output

Tạo artifact tổng hợp với:
1. **Overview** — mục tiêu, scope, complexity score
2. **Screen list** — tất cả screens + routes
3. **Component list** — tất cả components cần tạo
4. **API list** — tất cả endpoints
5. **Task breakdown** — ordered, có dependencies
6. **Risks** — blockers, mitigations
7. **Estimation** — thời gian ước tính

> Sau khi user approve → bắt đầu `/do-task` cho từng task trong breakdown.
