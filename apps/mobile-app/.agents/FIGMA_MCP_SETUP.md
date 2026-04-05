# Kết nối Figma MCP

Hướng dẫn kết nối Figma MCP để AI có thể đọc trực tiếp design từ Figma khi implement UI.

## Yêu cầu

- Tài khoản Figma (free plan trở lên)

## Setup (chỉ cần làm 1 lần)

### Bước 1: MCP config đã sẵn sàng

File `.agents/mcp.json` đã được cấu hình sẵn với Figma Remote MCP:

```json
{
  "mcpServers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp",
      "disabled": false,
      "autoApprove": ["get_design_context", "get_screenshot", "whoami"]
    }
  }
}
```

> Không cần cài thêm gì — Remote MCP hoạt động qua cloud của Figma.

### Bước 2: Xác thực Figma

Lần đầu tiên sử dụng Figma MCP, hệ thống sẽ:

1. **Popup trình duyệt** yêu cầu đăng nhập Figma
2. Đăng nhập bằng tài khoản Figma của bạn
3. **Cho phép (Authorize)** truy cập Figma
4. Quay lại IDE — kết nối sẵn sàng ✅

> Chỉ cần xác thực **1 lần**. Các lần sau sẽ tự kết nối.

### Bước 3: Verify kết nối

Kiểm tra MCP Servers trong IDE:
- Tìm mục **MCP SERVERS** (thường ở sidebar hoặc status bar)
- **figma** hiển thị trạng thái **Connected** ✅

Nếu IDE không tự đọc `.agents/mcp.json`, config thủ công:
1. Mở **Command Palette**: `Cmd + Shift + P`
2. Tìm: **"Open MCP config"** hoặc **"MCP settings"**
3. Thêm nội dung từ `.agents/mcp.json` vào

---

## Cách sử dụng

### Cung cấp Figma link trong chat

```
/do-task Tạo màn hình OrderList https://figma.com/design/abc123/MyApp?node-id=123-456

/new-screen OrderDetail https://figma.com/design/abc123/MyApp?node-id=789-012

/new-component OrderCard https://figma.com/design/abc123/MyApp?node-id=345-678
```

AI sẽ tự động:
1. Đọc design từ Figma qua MCP
2. Trích xuất colors, spacing, typography, layout
3. Map sang design tokens (`COLORS.*`, `SPACING.*`, `FONT_SIZE.*`)
4. Implement pixel-perfect theo design

### Lấy Figma link đúng cách

1. Mở file Figma trong trình duyệt
2. **Chọn frame/component** cần implement
3. Click phải → **Copy link to selection** (hoặc `Cmd + L`)
4. Paste link vào chat

> **Tip**: Chọn đúng frame cụ thể (ví dụ: "OrderCard") thay vì chọn cả page — AI sẽ đọc chính xác hơn.

---

## Troubleshooting

### ❌ "MCP server not connected"

**Cách fix**:
1. Kiểm tra `.agents/mcp.json` tồn tại và đúng format
2. Restart IDE
3. Nếu IDE yêu cầu config riêng → copy nội dung `.agents/mcp.json` vào MCP settings của IDE

### ❌ "Permission denied" hoặc "Unauthorized"

**Cách fix**:
- Kiểm tra bạn có **view access** vào file Figma đó
- Nếu file private → yêu cầu owner share cho bạn
- Thử xác thực lại: xóa token cũ và đăng nhập lại

### ❌ "Cannot read design data"

**Cách fix**:
- Chọn **frame cụ thể** trong Figma trước khi copy link
- Link nên có dạng: `...?node-id=123-456`
- Tránh copy link của cả page/file — quá rộng

### ❌ AI không sử dụng Figma data

**Cách fix**:
- Paste **trực tiếp** Figma link vào câu chat
- Hoặc nói rõ: "Implement theo design Figma: [link]"

---

## FAQ

**Q: Có miễn phí không?**
A: Có. Remote MCP hoạt động với tất cả Figma plans, kể cả free.

**Q: AI có thể sửa Figma design không?**
A: Không. MCP chỉ **đọc** design data. AI không thể sửa file Figma.

**Q: Có cần cài Figma Desktop app không?**
A: Không. Remote MCP chạy qua cloud, chỉ cần trình duyệt để đăng nhập.

**Q: Nếu dự án không dùng Figma thì sao?**
A: Các workflow vẫn hoạt động bình thường — bước Figma tự động bỏ qua nếu không có link.

**Q: Hỗ trợ những IDE nào?**
A: Bất kỳ IDE nào hỗ trợ MCP protocol — VS Code, Cursor, Kiro, Windsurf, v.v.
