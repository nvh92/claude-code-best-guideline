# Claude Code — Best Practices & Tips

<div align="center">

**Hướng dẫn thực chiến cho Claude Code CLI**

Tổng hợp từ Anthropic official docs, Boris (creator of Claude Code), và community patterns thực tế.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Language](https://img.shields.io/badge/Language-Vietnamese%20%7C%20English-blue)](#)

[🇻🇳 Tiếng Việt](#) · [🇬🇧 English](./README.md)

</div>

---

## Tài nguyên tham khảo

| Nguồn | Dùng khi |
|---|---|
| [Anthropic Official Docs](https://docs.anthropic.com/en/docs/claude-code) | Source of truth — đọc đầu tiên |
| [claude-plugins-official](https://github.com/anthropics/claude-plugins-official) | Tìm plugin cài ngay: `/plugin install <name>@claude-plugins-official` |
| [everything-claude-code](https://github.com/affaan-m/everything-claude-code) | Skills, hooks, commands, agents mẫu — copy & customize |
| [awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) | Thư viện 100+ subagent chuyên biệt |

---

## Mục lục

| Chủ đề | Tóm tắt |
|---|---|
| [1. Claude Code là gì?](#1-claude-code-là-gì) | Agent thực thi, không phải chatbot |
| [2. Bộ công cụ cốt lõi](#2-bộ-công-cụ-cốt-lõi) | Khi nào dùng cái gì |
| [3. CLAUDE.md](#3-claudemd--bộ-não-của-dự-án) | Memory của project — phân cấp & rules/ |
| [4. Skills](#4-skills--quy-trình-tái-sử-dụng) | Slash commands tái sử dụng |
| [5. Plugins](#5-plugins--skills-đóng-gói-sẵn) | Cài một lệnh, có ngay bộ tools |
| [6. Subagents](#6-subagents--delegation--song-song) | Delegation & parallel work |
| [7. Hooks](#7-hooks--tự-động-hóa-deterministic) | Automation không bao giờ bỏ qua |
| [8. Plan Mode](#8-plan-mode--thiết-kế-trước-khi-xây) | Explore → Plan → Implement |
| [9. Context Management & Session Handoff](#9-context-management--tài-nguyên-quý-nhất) | Giữ context sạch + không mất tiến độ |
| [10. MCP](#10-mcp--kết-nối-công-cụ-ngoài) | Kết nối GitHub, DB, Figma... |
| [11. Git Worktrees](#11-git-worktrees--song-song-an-toàn) | Nhiều AI sessions song song |
| [12. Prompting hiệu quả](#12-prompting-hiệu-quả) | Viết prompt ra kết quả tốt |
| [13. Anti-patterns](#13-anti-patterns-phổ-biến) | Những lỗi hay gặp & cách sửa |
| [14. Quick Reference](#14-quick-reference) | Bảng lệnh tra nhanh |

---

## 1. Claude Code là gì?

> Claude Code là **general agent** — không phải autocomplete, không phải chatbot. Nó đọc file, chạy lệnh, sửa code, quản lý git, kết nối dịch vụ ngoài, và làm việc tự động khi bạn vắng mặt.

### So sánh với chat thông thường

| Chat thường | Claude Code CLI |
|---|---|
| Trả lời câu hỏi | Thực thi hành động trong codebase |
| Mỗi cuộc trò chuyện độc lập | Session có context, resume được |
| Bạn copy-paste code | Claude đọc/viết file trực tiếp |
| Không biết repo của bạn | Hiểu toàn bộ cấu trúc project |
| Một luồng xử lý | Chạy song song nhiều sessions |

### Mô hình hoạt động

```
Bạn mô tả kết quả
    ↓
Claude explore + plan + implement
    ↓
Bạn review + redirect
```

---

## 2. Bộ công cụ cốt lõi

**Hiểu khi nào dùng cái gì quan trọng hơn biết cái gì tồn tại.**

```
┌─────────────────────────────────────────────────────────┐
│                    MCP Servers                          │
│         (Kết nối GitHub, DB, Figma, Notion...)         │
├─────────────────────────────────────────────────────────┤
│  CLAUDE.md  │  Skills  │  Subagents  │  Hooks          │
│  (Memory)   │  (Reuse) │  (Parallel) │  (Auto)         │
├─────────────────────────────────────────────────────────┤
│              Plan Mode + Context Management             │
└─────────────────────────────────────────────────────────┘
```

| Công cụ | Dùng khi | Ví dụ |
|---|---|---|
| **CLAUDE.md** | Cần Claude nhớ context mọi session | Stack, conventions, commands |
| **Skills** | Workflow lặp lại nhiều lần | `/new-feature`, `/deploy`, `/review` |
| **Plugins** | Cần workflow phổ biến, cài ngay | PR review, frontend design |
| **Subagents** | Task cần expertise riêng hoặc song song | Security review, test generation |
| **Hooks** | Hành động phải xảy ra mọi lúc, không exception | Lint sau edit, notify khi xong |
| **Plan Mode** | Task phức tạp, nhiều file, chưa rõ hướng | Refactor lớn, feature mới |
| **MCP** | Kết nối service bên ngoài | GitHub PRs, database queries |
| **Git Worktrees** | Nhiều AI task chạy song song | Feature A + Feature B cùng lúc |

---

## 3. CLAUDE.md — Bộ não của dự án

### Cơ chế tải tự động

CLAUDE.md được load **tự động** mỗi session. Hệ thống phân cấp từ global → local:

```
~/.claude/CLAUDE.md              ← Global (áp dụng mọi project)
~/.claude/rules/                 ← Global rules tách thành nhiều files nhỏ theo topic
    ├── code-style.md            ← Mỗi file = 1 chủ đề, load cùng lúc với CLAUDE.md
    ├── security.md
    └── workflow.md
project/CLAUDE.md                ← Project-wide (commit vào git)
project/CLAUDE.local.md          ← Cá nhân (gitignore, không commit)
project/front/CLAUDE.md          ← Load khi làm việc trong front/
project/back/CLAUDE.md           ← Load khi làm việc trong back/
```

**Dùng `~/.claude/rules/` khi nào?**

Thay vì nhét mọi thứ vào một file `~/.claude/CLAUDE.md` dài → tách thành files nhỏ theo chủ đề. Claude load tất cả files trong `rules/` tự động, không cần khai báo.

```
# Ví dụ cách tổ chức rules/
~/.claude/rules/
    ├── code-style.md     ← Quy tắc format, naming chung cho mọi project
    ├── security.md       ← Các rule bảo mật bắt buộc
    ├── git.md            ← Convention commit, branch, PR
    └── no-do.md          ← Những gì Claude không được làm (override default)
```

> Khi bạn làm việc với `front/src/components/`, Claude tự load: Global + rules/ → Project → `front/` → `front/src/components/` (nếu có).

### Nguyên tắc viết CLAUDE.md tốt

**Giữ ngắn — dưới 150 dòng cho root CLAUDE.md**

Claude Code's system prompt đã có ~50 instructions sẵn. CLAUDE.md quá dài → Claude bắt đầu bỏ qua rules ở giữa.

```markdown
# ✅ NÊN include — Claude không tự biết
- Bash commands để chạy project
- Code style khác default (tab thay space, line width 120)
- Branch naming convention của team
- Gotchas cụ thể của project này
- Required env vars để chạy local

# ❌ KHÔNG include — Claude đã biết hoặc không cần thiết
- "Write clean code"
- Standard language conventions
- Thông tin thay đổi thường xuyên
- File-by-file description của codebase
- Long explanations / tutorials
```

> **Tips tăng adherence:**
> - Dùng `IMPORTANT` và `YOU MUST` cho rules tuyệt đối
> - Test từng rule: nếu Claude đã làm đúng mà không cần → xóa đi
> - Dùng `@import` để modular hóa thay vì nhét tất cả vào một file
> - Check CLAUDE.md vào git để cả team đều được hưởng

### Template cơ bản

````markdown
# CLAUDE.md

## Project
[1-2 câu mô tả app là gì]

## Stack
[Tech stack ngắn gọn]

## Commands
```bash
npm run dev      # Start dev server
npm test         # Run tests
npm run lint     # Lint code
```

## Conventions
- [Chỉ những gì khác default]

## YOU MUST
- [Rule tuyệt đối không được vi phạm]
````

Xem thêm: [templates/CLAUDE.md.example](./templates/CLAUDE.md.example)

### Dùng `@import` để modular hóa

```markdown
# CLAUDE.md
See @README.md for project overview.

## Workflows
- Git: @docs/git-workflow.md
- Testing: @docs/testing-guide.md
```

---

## 4. Skills — Quy trình tái sử dụng

### Skills là gì

Skills = workflow đóng gói thành file `.claude/skills/<name>/SKILL.md`. Mỗi skill tự động có slash command `/name`.

> Kể từ 2026, skills và commands đã được hợp nhất — `.claude/commands/` vẫn hoạt động nhưng cách chuẩn là `.claude/skills/`.

### Frontmatter quan trọng

```yaml
---
name: fix-bug                    # → slash command /fix-bug
description: |                   # Claude dùng để auto-invoke (semantic matching)
  Fix bugs in the codebase.
  Use when: user reports a bug, test fails, error in logs
disable-model-invocation: true   # Claude KHÔNG tự gọi (dùng cho deploy, commit)
user-invocable: false            # Ẩn khỏi / menu (background knowledge)
allowed-tools: Bash(pytest *), Read, Grep
context: fork                    # Chạy trong subagent riêng
agent: Explore                   # Loại subagent khi context: fork
model: sonnet                    # Override model cho skill này
argument-hint: "<issue-id>"      # Gợi ý khi autocomplete
---
```

### 3 loại skills

**Loại 1: Auto-invoked** — Claude tự gọi khi task phù hợp

```yaml
---
name: api-conventions
description: REST API conventions cho project này. Dùng khi tạo hoặc sửa API endpoint.
user-invocable: false
---
# API Conventions
- Dùng kebab-case cho URL
- camelCase cho JSON fields
- Mọi endpoint phải có operation_id
```

**Loại 2: User-invoked** — bạn gọi thủ công

```yaml
---
name: deploy
description: Deploy lên production
disable-model-invocation: true
allowed-tools: Bash(*)
---
Deploy $ARGUMENTS:
1. Chạy full test suite
2. Build Docker image
3. Push và deploy lên cluster
4. Health check — rollback nếu fail
```

**Loại 3: Subagent skill** — chạy trong context riêng

```yaml
---
name: security-review
description: Review code cho security issues
context: fork
agent: Explore
allowed-tools: Read, Grep, Glob
---
Review $ARGUMENTS cho security vulnerabilities.
Report: file:line, severity (Critical/High/Medium/Low), suggested fix.
```

### Best practices

- Description phải chứa keywords người dùng sẽ nói → Claude dùng semantic matching để auto-invoke
- Dưới 500 dòng cho SKILL.md, tài liệu chi tiết đặt file riêng và `@reference`
- `$ARGUMENTS` cho toàn bộ args, `$0 $1 $2` cho positional args
- Whitelist lệnh cụ thể: `allowed-tools: Bash(pytest *)` thay vì toàn bộ `Bash(*)`

Xem thêm: [templates/skill.md.example](./templates/skill.md.example) · [everything-claude-code/.claude/](https://github.com/affaan-m/everything-claude-code/tree/main/.claude)

---

## 5. Plugins — Skills đóng gói sẵn

> Plugin = bundle gồm skills + hooks + agents + MCP servers. Cài một lệnh là có hết.

### Cài plugin

```bash
# Duyệt marketplace
/plugin

# Cài plugin cụ thể
/plugin install frontend-design@claude-plugins-official
/plugin install pr-review-toolkit@claude-plugins-official
/plugin install feature-dev@claude-plugins-official
```

### Plugins nổi bật

| Plugin | Dùng khi |
|---|---|
| **frontend-design** | Build UI — tự động invoke khi làm frontend, tránh "AI slop" aesthetics |
| **feature-dev** | Guided feature workflow với 3 agents: code-explorer, code-architect, code-reviewer |
| **pr-review-toolkit** | Review PR toàn diện: comments, tests, error handling, types |
| **code-review** | 5 Sonnet agents song song: CLAUDE.md compliance, bug detection, PR history |
| **aws-deploy** | Deploy lên AWS với architecture recommendations |
| **database** | Schema design, migrations, query optimization |

### Khi nào dùng Plugin vs tự viết Skill

```
Dùng Plugin khi:                  Tự viết Skill khi:
✅ Workflow phổ biến               ✅ Convention riêng của project
✅ Cần ngay, không muốn setup     ✅ Tích hợp với tools nội bộ
✅ Có sẵn trong marketplace       ✅ Business logic đặc thù
                                   ✅ Không có plugin phù hợp
```

---

## 6. Subagents — Delegation & Song song

### Tại sao cần Subagents

**Context poisoning:** Khi Claude research/explore codebase, nó đọc hàng trăm file → context window đầy với thông tin không liên quan → performance giảm đáng kể.

Subagent chạy trong **context window riêng biệt**. Main conversation giữ sạch.

### Built-in subagents (không cần config)

| Agent | Model | Tools | Dùng cho |
|---|---|---|---|
| **Explore** | Haiku (nhanh) | Read-only | Research, investigation |
| **Plan** | Sonnet | Read-only | Architecture decisions |
| **General** | Sonnet | Full access | Implementation tasks |

### Custom subagent

```markdown
---
name: security-auditor
description: Analyzes code for OWASP Top 10 vulnerabilities
tools: Read, Grep, Bash
model: sonnet
---

Bạn là security engineer chuyên OWASP.
Review: XSS, SQL injection, CSRF, auth bypass, data exposure.
Format: Critical/High/Medium/Low với file:line và suggested fix.
```

Đặt tại: `.claude/agents/security-auditor.md`

### Worktree Isolation (2026)

```yaml
isolation: worktree   # Subagent làm việc trên git worktree riêng
```

Nhiều subagents có thể edit files song song mà không conflict. Worktree tự cleanup nếu không có changes.

### Patterns hiệu quả

**Writer/Reviewer pattern — context sạch, review không bị bias:**

```
Session A: Implement rate limiter
Session B: Review rate limiter implementation
```

**Parallel analysis:**

```
"Dùng subagents song song:
 - Subagent 1: Review security của auth module
 - Subagent 2: Generate tests cho auth module
 Report kết quả về main session."
```

**Investigation trước implementation:**

```
"Dùng subagent Explore để research codebase,
 tìm patterns hiện tại trong back/app/services/.
 Chỉ summary lại — không làm bẩn main context."
```

---

## 7. Hooks — Tự động hóa deterministic

### Hooks vs CLAUDE.md rules

| | CLAUDE.md rules | Hooks |
|---|---|---|
| Tính chất | Advisory | Deterministic |
| Có thể bị bỏ qua | Có | Không |
| Cấu hình ở | CLAUDE.md | `.claude/settings.json` |
| Dùng khi | Convention, style | Bắt buộc phải chạy |

### Lifecycle events

```
PreToolUse    → Trước khi Claude dùng tool (có thể block action)
PostToolUse   → Sau khi tool chạy xong
Notification  → Khi Claude muốn thông báo với user
Stop          → Khi Claude kết thúc task
```

### Cấu hình trong `.claude/settings.json`

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{
          "type": "command",
          "command": "pnpm lint --fix 2>/dev/null; true"
        }]
      }
    ],
    "Stop": [
      {
        "hooks": [{
          "type": "command",
          "command": "osascript -e 'display notification \"Claude xong việc\" with title \"Claude Code\"'"
        }]
      }
    ]
  }
}
```

### Use cases thực tế

| Hook | Trigger | Lệnh |
|---|---|---|
| Auto lint | Sau mỗi Edit/Write | `ruff check . --fix` |
| Auto format | Sau mỗi Edit/Write | `prettier --write` |
| Desktop notify | Khi Stop | `osascript notification` |
| Log session | Khi Start | Script ghi timestamp |
| Block nguy hiểm | PreToolUse | Kiểm tra và exit nếu nguy hiểm |

> **Tip:** Dùng AI để viết hooks: `"Viết hook chạy eslint sau mỗi lần edit file TypeScript."`

---

## 8. Plan Mode — Thiết kế trước khi xây

### Kích hoạt

```bash
claude --permission-mode plan     # Bắt đầu session ở Plan Mode
Shift+Tab (2 lần)                 # Toggle trong session đang chạy
```

Trong Plan Mode: Claude chỉ **đọc** (Read, Grep, Glob) — không edit, không chạy lệnh có side effect.

### Workflow 4 bước

```
1. EXPLORE  (Plan Mode)
   Claude đọc codebase, hiểu structure
   → "Đọc @back/app/ và hiểu layered architecture hiện tại"

         ↓

2. PLAN  (Plan Mode)
   Claude tạo implementation plan chi tiết
   → "Tôi muốn thêm OAuth. List tất cả files cần thay đổi và thứ tự."
   → Ctrl+G để mở và edit plan trong editor trước khi proceed

         ↓

3. IMPLEMENT  (Normal Mode)
   Switch sang Normal Mode, bắt đầu thực thi
   → "Implement theo plan. Verify sau mỗi bước."

         ↓

4. COMMIT
   → "Commit với message mô tả đầy đủ và tạo PR"
```

### Khi nào dùng Plan Mode

```
✅ Task sửa 3+ files
✅ Chưa quen với phần codebase đang sửa
✅ Feature mới phức tạp
✅ Refactor lớn

❌ Fix typo, rename variable
❌ Task nhỏ rõ ràng có thể mô tả trong 1 câu
❌ Add log line
```

### Thinking Mode (Extended Thinking)

```bash
claude --thinking          # Bật extended thinking
/thinking on               # Toggle trong session
```

Dùng cho task đặc biệt phức tạp cần deep reasoning — không phải mọi task.

---

## 9. Context Management — Tài nguyên quý nhất

### Tại sao quan trọng

Context window chứa: toàn bộ conversation + mỗi file Claude đọc + mỗi command output. Khi đầy, Claude bắt đầu "quên" instructions và mắc nhiều lỗi hơn.

### Các lệnh quản lý context

```bash
/clear                          # Reset toàn bộ context
/context                        # Xem % context window đang dùng
/compact                        # Nén context, giữ lại phần quan trọng (khi context window 70-80%)
/compact focus on API changes   # Nén với instruction cụ thể
/btw <câu hỏi>                  # Hỏi nhanh — KHÔNG vào context, không tốn token
Esc                             # Stop Claude ngay, giữ context
Esc+Esc  /  /rewind             # Mở rewind menu, khôi phục checkpoint trước
```

### Rules quản lý context

```
✅ /clear sau mỗi feature hoàn chỉnh
✅ /clear khi chuyển task loại khác (backend → frontend)
✅ /clear sau khi sửa cùng lỗi 2 lần vẫn sai
✅ Dùng subagent cho investigation → không tốn main context
✅ /rename <tên> để đặt tên session, dễ resume sau

❌ Không /clear giữa chừng khi task chưa xong
❌ Không để một session chứa quá nhiều loại task khác nhau
```

### Resume session

```bash
claude --continue       # Tiếp tục session gần nhất
claude --resume         # Chọn từ danh sách session có tên
```

### Checkpoints

Claude tự động checkpoint trước mỗi thay đổi. Dùng `Esc+Esc` → rewind menu để:
- Restore conversation only
- Restore code only
- Restore cả hai

---

### Session Handoff — Vòng lặp không mất context

> Claude không có memory giữa các session. Session mới bắt đầu từ zero → mất thời gian re-orient, dễ đi sai hướng.

**Giải pháp:** Dùng CLAUDE.md làm "bộ nhớ dài hạn" — cập nhật cuối mỗi session, đọc đầu mỗi session mới.

```
Kết thúc session → Update CLAUDE.md
                         ↓
Bắt đầu session mới → Claude đọc CLAUDE.md → Tiếp tục đúng chỗ
```

**Bước 1 — Kết thúc session đúng cách**

```
"Trước khi kết thúc, cập nhật CLAUDE.md với:
 - Những gì đã làm xong
 - Trạng thái hiện tại của từng phần
 - Bước tiếp theo cần làm
 - Quyết định quan trọng & lý do"
```

**Bước 2 — Bắt đầu session mới đúng cách**

```
"Đọc CLAUDE.md cho tôi tóm tắt:
 Dự án đang ở đâu, bước tiếp theo là gì,
 và có gì tôi cần chú ý không?"
```

**Bước 3 — Plan Mode cho session lớn (> 15 phút)**

```
"Đọc CLAUDE.md và tóm tắt hiện trạng project,
 liệt kê những gì sẽ làm trong session hôm nay,
 ưu tiên những gì và theo thứ tự nào?"
```

**Gợi ý section trong CLAUDE.md để hỗ trợ handoff:**

```markdown
## Current State
<!-- Claude cập nhật cuối mỗi session -->
- **Last session:** [ngày] — [những gì đã xong]
- **In progress:** [feature/task đang làm dở]
- **Next:** [bước tiếp theo cụ thể]

## Key Decisions
<!-- Không xóa — append khi có decision mới -->
- [Decision] — [lý do] — [ngày]
```

> **Tip:** Dành 2 phút cuối mỗi session update CLAUDE.md → tiết kiệm 10 phút đầu session tiếp theo. Xem chi tiết: [docs/vi/session-handoff.md](./docs/vi/session-handoff.md)

---

## 10. MCP — Kết nối công cụ ngoài

### MCP là gì

Model Context Protocol = universal adapter kết nối Claude với external services. Mỗi MCP server expose tools, resources, prompts thành slash commands.

### Cài đặt

```bash
claude mcp add playwright npx @playwright/mcp@latest
claude mcp add github gh auth login
```

### Deferred Tool Loading (2026)

Claude Code không load đầy đủ tool schemas khi start — chỉ load tên, fetch schema khi cần. Giảm context overhead đáng kể khi dùng 50+ MCP tools.

### Các MCP server hữu ích

| Server | Dùng cho |
|---|---|
| **GitHub MCP** | Tạo PR, đọc issues, review comments |
| **Playwright MCP** | UI testing tự động |
| **Database MCP** | Query DB trực tiếp từ Claude |
| **Figma MCP** | Đọc design, implement từ Figma |
| **Notion MCP** | Đọc/viết tasks và documents |

> **Tip:**
> - Cài gh CLI để Claude dùng GitHub API hiệu quả hơn
> - Claude tự học CLI mới: "Dùng foo-cli --help để học cách dùng, rồi giải quyết X."
> - Dùng /permissions để allowlist domains Claude được phép fetch


---

## 11. Git Worktrees — Song song an toàn

### Vấn đề

Nhiều Claude sessions cùng sửa một thư mục → conflict. Git worktrees tạo **bản sao working tree riêng biệt** cho mỗi branch → sessions làm việc độc lập.

### Setup cơ bản

```bash
# Tạo worktrees
git worktree add ../my-project-feature-a feat/feature-a
git worktree add ../my-project-feature-b feat/feature-b

# Chạy Claude Code trong từng worktree
cd ../my-project-feature-a && claude   # Terminal 1
cd ../my-project-feature-b && claude   # Terminal 2
```

### Subagent worktrees (tự động)

```yaml
# .claude/agents/parallel-worker.md
---
isolation: worktree
---
```

Subagent tự tạo worktree riêng, làm việc độc lập, cleanup khi xong.

### Pattern thực tế

```
Worktree 1: Develop feature-auth
Worktree 2: Review + fix bug production
Worktree 3: Refactor module cũ
→ 3 Claude sessions chạy song song, không conflict
```

---

## 12. Prompting hiệu quả

### Nguyên tắc cơ bản

**Mô tả outcome, không micromanage cách làm:**

```
❌ "Mở file users.py, tìm class User, thêm method validate_email..."
✅ "Thêm email validation vào User model. Chạy tests sau khi xong."
```

**Cung cấp verification criteria:**

```
❌ "Implement email validation"
✅ "Implement email validation. Test cases:
    user@example.com → True
    invalid → False
    user@.com → False
    Chạy tests và confirm xanh hết."
```

**Reference bằng `@`:**

```
✅ "Follow pattern tương tự @back/app/services/user_service.py"
✅ "Đọc @SPEC.md trước khi implement"
✅ "Schema tại @back/app/schemas/user.py"
```

**Scope rõ ràng:**

```
❌ "Fix the login bug"

✅ "User report login fail sau session timeout.
    Check auth flow tại src/auth/, đặc biệt token refresh.
    Viết failing test reproduce issue, rồi fix."
```

### Để Claude phỏng vấn bạn khi task lớn/mơ hồ

```
Tôi muốn build [mô tả ngắn].

Interview tôi chi tiết bằng AskUserQuestion tool.
Hỏi về: implementation, UX, edge cases, tradeoffs, security.
Đừng hỏi điều hiển nhiên — đào sâu vào phần khó tôi chưa nghĩ tới.
Phỏng vấn đến khi cover hết, rồi viết spec ra SPEC.md.
```

### Course-correct sớm

```
Esc          → Stop ngay khi thấy sai hướng
Esc+Esc      → Rewind về checkpoint tốt gần nhất

Sau 2 lần sửa cùng lỗi vẫn sai:
  → /clear + prompt lại tốt hơn (đừng tiếp tục correction loop)

Vague prompt OK khi đang explore:
  → "What would you improve in this file?"
```

---

## 13. Anti-patterns phổ biến

### Kitchen sink session

```
❌ Một session: debug bug → thêm feature → refactor → setup CI/CD
   → Context ngập rác, performance giảm

✅ Fix: /clear giữa các task khác loại
```

### Infinite correction loop

```
❌ Claude sai → sửa → vẫn sai → sửa lại → context đầy failed attempts

✅ Fix: Sau 2 lần vẫn sai → /clear, viết prompt tốt hơn dựa trên những gì học được
```

### Over-specified CLAUDE.md

```
❌ CLAUDE.md > 150 dòng → Claude bỏ qua rules ở giữa

✅ Fix: Prune ruthlessly. Dùng Skills/Subagents cho domain knowledge đặc thù.
```

### Trust-then-verify gap

```
❌ Claude tạo code trông đúng nhưng không handle edge cases → ship ngay

✅ Fix: Luôn có verification (tests, scripts, screenshots).
        Không verify = không ship.
```

### Infinite exploration

```
❌ "Investigate toàn bộ codebase" → Claude đọc hàng trăm file → context đầy

✅ Fix: Scope investigation hẹp hoặc dùng subagent Explore
```

### Không dùng Plan Mode cho task phức tạp

```
❌ Code ngay mà không explore → giải quyết sai problem, phải làm lại

✅ Fix: Plan Mode cho task sửa 3+ files hoặc chưa quen code area
```

### Merge AI output không review

```
❌ Claude không biết business context.
   Output trông đúng về syntax nhưng sai về logic.

✅ Fix: Luôn đọc diff trước khi commit. Human gate không được bỏ qua.
```

---

## 14. Quick Reference

### Session commands

```bash
# Khởi động
claude                              # Session mới
claude --continue                   # Tiếp tục session gần nhất
claude --resume                     # Chọn session để resume
claude --permission-mode plan       # Bắt đầu ở Plan Mode

# Trong session
/clear                              # Reset context
/context                            # Xem context usage (%)
/compact                            # Nén context
/compact focus on API changes       # Nén có chủ đích
/btw <câu hỏi>                      # Hỏi nhanh không tốn context
/rename <tên>                       # Đặt tên session
/agents                             # Xem/tạo subagents
/rewind                             # Mở checkpoint menu
/thinking on                        # Bật extended thinking

# Keyboard shortcuts
Esc                                 # Stop Claude
Esc+Esc                             # Stop + rewind
Shift+Tab                           # Chuyển permission mode
Ctrl+G                              # Mở plan trong editor
```

### Non-interactive (CI/scripts)

```bash
claude -p "prompt"                              # Chạy một lần
claude -p "..." --output-format json            # Structured output
claude --permission-mode auto -p "fix lint"     # Auto mode
```

### Plugin commands

```bash
/plugin                             # Duyệt marketplace
/plugin install <name>@claude-plugins-official
```

### MCP

```bash
claude mcp add <name> <command>
claude mcp list
claude mcp remove <name>
```

---

## Đóng góp

Mọi đóng góp đều được chào đón! Tạo issue hoặc pull request nếu bạn có tip hữu ích, pattern thực tế, hoặc phát hiện thông tin lỗi thời.

---

<div align="center">

Made with ❤️ for the Vietnamese developer community

[🇬🇧 Read in English](./README.md)

</div>
