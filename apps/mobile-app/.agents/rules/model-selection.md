# Model Selection Guide

> Tham khảo bảng này để khuyến nghị model phù hợp nhất cho mỗi task.
> Luôn đưa ra khuyến nghị model ở **đầu mỗi task** trước khi bắt đầu làm việc.

## Bảng tra cứu nhanh

| Task | Model khuyến nghị | Lý do |
|------|-------------------|-------|
| **Đơn giản** (typo, format, boilerplate, i18n) | ⚡ Gemini Flash / Sonnet 4.6 | Nhanh, rẻ, đủ dùng |
| **Tạo component UI** | 🏆 Claude Sonnet 4.6 | Code sạch, convention tốt |
| **Implement feature mới** | 🏆 Claude Sonnet 4.6 | Balance quality/speed/cost |
| **Fix bug thường** | 🏆 Claude Sonnet 4.6 | Trace lỗi tốt, nhanh |
| **Viết unit test** | 🏆 Claude Sonnet 4.6 | Output test chất lượng |
| **Code review PR** | 🏆 Claude Sonnet 4.6 | Phát hiện pattern tốt |
| **Đọc Figma screenshot → Code** | 📚 Gemini 3.1 Pro (MEDIUM) | Multimodal, vision tốt |
| **Phân tích screenshot/video bug** | 📚 Gemini 3.1 Pro (MEDIUM) | Multimodal native |
| **Đọc/phân tích codebase lớn** | 📚 Gemini 3.1 Pro (LOW–MEDIUM) | Context 1M tokens, giá tốt |
| **Đọc tài liệu API dài** | 📚 Gemini 3.1 Pro (LOW) | Long context, rẻ |
| **Batch processing / data migration** | 📚 Gemini 3.1 Pro (Batch API) | 50% off, volume lớn |
| **Refactor lớn (10+ files)** | 🧠 Claude Opus 4.6 (Thinking) | Multi-file coherence |
| **Thiết kế architecture** | 🧠 Claude Opus 4.6 (Thinking) | Deep reasoning |
| **Security audit** | 🧠 Claude Opus 4.6 (Thinking) | Phát hiện vulnerability |
| **Debug production (crash, race condition)** | 🧠 Claude Opus 4.6 (Thinking) | Trace logic sâu |
| **Expo/SDK upgrade major** | 🧠 Claude Opus 4.6 (Thinking) | Breaking changes, multi-step |
| **Performance optimization** | 🧠 Claude Opus 4.6 (Thinking) | Memory, render cycle analysis |
| **Database schema design** | 🧠 Claude Opus 4.6 (Thinking) | Cần suy luận quan hệ phức tạp |

## Cách phân loại nhanh

```
Task vào → Hỏi 3 câu:

1. Có cần xử lý ảnh/video/file không?
   → Có → Gemini 3.1 Pro

2. Có phức tạp (10+ files, architecture, security)?
   → Có → Claude Opus 4.6 (Thinking)

3. Còn lại?
   → Claude Sonnet 4.6 (default)
```

## Format khuyến nghị

Khi bắt đầu mỗi task, output theo format sau:

```
💡 **Khuyến nghị model:** [Tên model]
📋 **Loại task:** [Đơn giản / Trung bình / Phức tạp]
💰 **Ước tính chi phí:** [$ Rẻ / $$ Vừa / $$$ Đắt]
⚡ **Tốc độ:** [Nhanh / Vừa / Chậm]
🔄 **Model hiện tại:** [Model đang dùng]
➡️ **Cần đổi không:** [Có — nên chuyển sang X / Không — đang dùng đúng]
```
