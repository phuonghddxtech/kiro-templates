# Kịch Bản Trình Diễn Toàn Diện (End-to-End Master Demo)

Đây là kịch bản demo trình diễn toàn bộ sinh thái của hệ thống Antigravity Agent dành cho Mobile Developer, trải dài mọi ngóc ngách của quá trình phát triển dự án.

## 🌟 Chân dung Kịch bản: Tính năng "Giỏ Hàng" (Cart Screen)
- **Idea**: Thêm màn hình Giỏ Hàng cho ứng dụng E-commerce (Hiển thị list các món hàng, tổng tiền, nút thanh toán).
- **Trạng thái**: Đã có file thiết kế Figma.

---

### Bước 1: Ý Tưởng & Phân Tích (Idea → Tasks)
*Ngữ cảnh: PM vừa chốt yêu cầu xây dựng màn hình Giỏ hàng. Dev sẽ bắt đầu bằng việc nhờ AI "nhai" specs và biến nó thành Tasks.*

**Lệnh thực hiện:**
```bash
/analyze-spec Xây dựng màn hình Giỏ hàng. Yêu cầu:
- Hiển thị danh sách sản phẩm (có nút xoá, đổi số lượng).
- Hiển thị tổng tiền.
- Bấm nút Checkout.
- Có link Figma: https://figma.com/design/xxxxx?node-id=CART
```

**Phản ứng của AI:**
- Bóc tách thành 4 Task nhỏ: 
  1. Tạo `CartService` (gọi API).
  2. Tạo `useCartStore` (Dùng Zustand để gom state offline).
  3. Tạo `CartItem` (Component giao diện từ Figma).
  4. Tạo `CartScreen` (Màn hình chính).

---

### Bước 2: Setup & Tích Hợp Figma (Figma MCP)
*Ngữ cảnh: Đi thẳng vào code phần giao diện trước. Dev sẽ kêu AI gõ Component `CartItem` cho giống hệt Figma.*

**Lệnh thực hiện:**
```bash
/new-component CartItem Tham chiếu thiết kế từ node Figma này: https://figma.com/design/xxxxx?node-id=CART_ITEM
```

**Phản ứng của AI:**
- Kết nối thông qua **Figma MCP**, đọc mã màu (Ví dụ: `Red-500` -> `COLORS.danger`), đọc spacing (`16px` -> `SPACING.md`).
- Tự động sinh ra file `CartItem.tsx` bằng React Native (StyleSheet bám chuẩn Rule 17).
- Chèn sẵn Test cơ bản cho accessibility.

---

### Bước 3: Lắp ráp Logic Toàn Tuyến (Init Source & Do Task)
*Ngữ cảnh: Nối UI vào Logic và tạo các Service.*

**Lệnh thực hiện:**
```bash
/do-task Thực hiện Task 1 và Task 2: Dựng CartService dùng Axios và useCartStore dùng Zustand có tích hợp bộ nhớ offline MMKV.
```

**Phản ứng của AI:**
- Bám sát `rules/state-management.md` & `rules/offline-caching.md`: Sinh ra file store của Zustand với `persist` middleware trỏ vào MMKV Storage.
- Tuân thủ `rules/api-integration.md`: Tạo `CartService.ts` bằng Axios.

---

### Bước 4: Tạo Screen (Ráp mọi thứ lại)
*Ngữ cảnh: Giao diện và API đã có, giờ tạo Màn Hình tổng.*

**Lệnh thực hiện:**
```bash
/new-screen CartScreen Ráp CartItem và bộ store Zustand vừa tạo vào màn hình này nhé.
```

**Phản ứng của AI:**
- Bám sát `rules/error-handling.md`: Sinh ra `CartScreen.tsx` với đủ 4 trạng thái bắt buộc: `Loading` (Skeleton), `Empty` (Chưa có gì trong giỏ hàng), `Error`, và hiển thị dữ liệu `Data`.
- Dùng FlatList tối ưu theo rule `performance.md`.

---

### Bước 5: Bắt Bug Thực Tế (Fix Bug)
*Ngữ cảnh: Khi chạy app, nút "Xoá món hàng" bấm trên Android khó ăn vì dính viền.*

**Lệnh thực hiện:**
```bash
/mobile-fix-issue Nút xoá trên Android bấm rất khó ăn, vùng chạm đang bị quá bé hụt tay.
```

**Phản ứng của AI:**
- Nhớ tới Rule `rules/accessibility.md`.
- Phát hiện ra lỗi `hitSlop`.
- Phản hồi: Tự động chèn `<TouchableOpacity hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}>` để nới rộng vùng thao tác trên Mobile an toàn mà không làm lỡ layout.

---

### Bước 6: Kỷ luật Thép — Ép viết Test (Write Unit Test)
*Ngữ cảnh: Dev lười viết Test và muốn trốn khâu Unit Test cho `useCartStore`.*

**Luật ngầm hoạt động:** 
AI chủ động nhắc nhở dựa trên bộ workflows: "Bạn chưa cung cấp Unit test cho store này. Hãy dùng `/write-tests` trước khi commit!".

**Lệnh thực hiện:**
```bash
/write-tests Hãy generate Unit test coverage cho file useCartStore.ts
```

**Phản ứng của AI:**
- Import thư viện `@testing-library/react-native`.
- Sinh ra full kịch bản test: "Trường hợp thêm đồ vào giỏ", "Trường hợp xoá đồ", bảo đảm **bao phủ 80%** (theo Rule Testing).

---

### Bước 7: Done! Push Code (Commit Push)
*Ngữ cảnh: Mọi chức năng đã hoàn thành trơn tru.*

**Lệnh thực hiện:**
```bash
/commit-push Màn hình giỏ hàng
```

**Phản ứng của AI:**
- Mở quy trình tự xét duyệt: Chạy `eslint`, chạy `jest` xem test vừa gõ có pass không.
- Nếu Pass: AI tự động parse nội dung, tuân thủ độ dài commit < 60 ký tự.
- Thực thi: `git add .`, `git commit -m "feat(cart): implement cart screen with zustand and offline"`, và `git push origin`.

---
🎉 **HOÀN TẤT VÒNG ĐỜI**: Tất cả đều được tự động hoá và bám sát cực độ vào cuốn sổ "17 Lời Răn Mobile Developer" của bạn.
