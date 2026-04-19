# Session Handoff — Vòng lặp không mất context

> Workflow giữ context liên tục giữa các session — không bao giờ mất tiến độ.

---

## Vấn đề

Mỗi khi kết thúc session Claude, toàn bộ context bị xóa. Session mới bắt đầu từ zero.

**Hậu quả thực tế:**
- Mất thời gian re-orient, giải thích lại project mỗi session
- Dễ đi sai hướng vì Claude không biết context
- Quyết định kỹ thuật bị "quên" và phải make lại
- Progress không rõ ràng — task nào xong, task nào chưa

**Giải pháp:** Dùng CLAUDE.md làm "bộ nhớ dài hạn" — cập nhật cuối mỗi session, đọc đầu mỗi session mới.

---

## Vòng lặp Session Handoff

```
Kết thúc session → Update CLAUDE.md
                          ↓
Bắt đầu session mới → Claude đọc CLAUDE.md → Tiếp tục đúng chỗ
```

---

## Bước 1 — Kết thúc session đúng cách

**Trước khi đóng terminal, yêu cầu Claude cập nhật CLAUDE.md:**

```
"Trước khi kết thúc, cập nhật CLAUDE.md với:
 - Những gì đã làm xong hôm nay
 - Trạng thái hiện tại của từng phần đang làm
 - Bước tiếp theo cần làm
 - Quyết định kỹ thuật quan trọng & lý do"
```

**Hoặc ngắn gọn hơn:**

```
"Update CLAUDE.md: progress hôm nay, next steps, decisions made."
```

**Gợi ý section trong CLAUDE.md:**

```markdown
## Current State
<!-- Claude cập nhật cuối mỗi session -->
- **Last session:** [ngày] — [những gì đã xong]
- **In progress:** [feature/task đang làm dở]
- **Next:** [bước tiếp theo cụ thể]

## Key Decisions
- [Decision 1] — [lý do]
- [Decision 2] — [lý do]
```

---

## Bước 2 — Bắt đầu session mới đúng cách

**Thay vì lao vào task ngay, để Claude đọc context trước:**

```
"Đọc CLAUDE.md cho tôi tóm tắt:
 Dự án đang ở đâu,
 Bước tiếp theo là gì,
 và có gì tôi cần chú ý không?"
```

**Kết hợp với Plans (nếu có):**

```
"Đọc CLAUDE.md và @plans/feat-auth-oauth.md.
 Tóm tắt tình trạng, rồi tiếp tục task tiếp theo."
```

---

## Bước 3 — Plan Mode cho session lớn

**Khi session dự kiến > 15 phút → dùng Plan Mode để lập kế hoạch trước:**

```bash
# Khởi động ở Plan Mode
claude --permission-mode plan

# Hoặc toggle trong session
Shift+Tab (2 lần)
```

**Prompt để plan session:**

```
"Đọc CLAUDE.md và tóm tắt hiện trạng project.
 Liệt kê những gì sẽ làm trong session hôm nay.
 Ưu tiên những gì và theo thứ tự nào?"
```

**Sau khi có plan, switch sang Normal Mode và implement:**

```bash
Shift+Tab   # Toggle về Normal Mode
```

---

## Template đầy đủ cho CLAUDE.md với Session Handoff

````markdown
# CLAUDE.md

## Project
[Mô tả ngắn]

## Stack
[Tech stack]

## Commands
```bash
[các lệnh quan trọng]
```

## Architecture
[Quyết định kiến trúc cần nhớ]

## YOU MUST
[Rules tuyệt đối]

---

## Current State
<!-- Cập nhật cuối mỗi session -->
- **Last updated:** [ngày]
- **Last session:** [những gì đã làm]
- **In progress:** [đang làm gì]
- **Next steps:**
  1. [step 1]
  2. [step 2]

## Key Decisions
<!-- Không xóa — thêm vào khi có quyết định mới -->
- [Decision] — [lý do] — [ngày]
````

---

## Tips nâng cao

**Dùng `/rename` để dễ resume:**

```bash
/rename oauth-feature-session-3
```

**Resume đúng session:**

```bash
claude --resume   # Chọn từ danh sách
```

**Kết hợp Plans/ để track chi tiết hơn:**

Plans/ dùng cho task-level tracking (tick [x] từng task).
CLAUDE.md dùng cho session-level context (tình trạng tổng quan).

Dùng cả hai cho project lớn.

---

> **Takeaway:** Dành 2 phút cuối mỗi session để update CLAUDE.md. Tiết kiệm 10 phút đầu mỗi session tiếp theo.
