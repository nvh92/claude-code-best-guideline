# Claude Code — Best Practices & Tips

<div align="center">

**A practical guide to Claude Code CLI**

Compiled from Anthropic official docs, Boris (creator of Claude Code), and real-world community patterns.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Language](https://img.shields.io/badge/Language-Vietnamese%20%7C%20English-blue)](#)

[🇻🇳 Tiếng Việt](./README.vi.md) · [🇬🇧 English](#)

</div>

---

## Key Resources

| Source | When to use |
|---|---|
| [Anthropic Official Docs](https://docs.anthropic.com/en/docs/claude-code) | Source of truth — read first |
| [claude-plugins-official](https://github.com/anthropics/claude-plugins-official) | Find ready-to-install plugins: `/plugin install <name>@claude-plugins-official` |
| [everything-claude-code](https://github.com/affaan-m/everything-claude-code) | Sample skills, hooks, commands, agents — copy & customize |
| [awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) | Library of 100+ specialized subagents |

---

## Table of Contents

| Topic | Summary |
|---|---|
| [1. What is Claude Code?](#1-what-is-claude-code) | An execution agent, not a chatbot |
| [2. Core Toolkit](#2-core-toolkit) | When to use what |
| [3. CLAUDE.md](#3-claudemd--the-projects-brain) | Project memory — hierarchy & rules/ |
| [4. Skills](#4-skills--reusable-workflows) | Reusable slash commands |
| [5. Plugins](#5-plugins--pre-packaged-skills) | Install once, get everything |
| [6. Subagents](#6-subagents--delegation--parallelism) | Delegation & parallel work |
| [7. Hooks](#7-hooks--deterministic-automation) | Automation that never gets skipped |
| [8. Plan Mode](#8-plan-mode--design-before-you-build) | Explore → Plan → Implement |
| [9. Context Management & Session Handoff](#9-context-management--your-most-precious-resource) | Keep context clean + never lose progress |
| [10. MCP](#10-mcp--connecting-external-tools) | Connect GitHub, DB, Figma... |
| [11. Git Worktrees](#11-git-worktrees--safe-parallelism) | Multiple AI sessions in parallel |
| [12. Effective Prompting](#12-effective-prompting) | Write prompts that get good results |
| [13. Anti-patterns](#13-common-anti-patterns) | Common mistakes and how to fix them |
| [14. Quick Reference](#14-quick-reference) | Command cheat sheet |

---

## 1. What is Claude Code?

> Claude Code is a **general agent** — not autocomplete, not a chatbot. It reads files, runs commands, edits code, manages git, connects to external services, and works autonomously while you're away.

### Comparison with regular chat

| Regular Chat | Claude Code CLI |
|---|---|
| Answers questions | Executes actions in the codebase |
| Each conversation is independent | Sessions have context, resumable |
| You copy-paste code | Claude reads/writes files directly |
| Doesn't know your repo | Understands the full project structure |
| Single thread | Can run multiple sessions in parallel |

### How it works

```
You describe the outcome
    ↓
Claude explores + plans + implements
    ↓
You review + redirect
```

---

## 2. Core Toolkit

**Understanding when to use what matters more than knowing what exists.**

```
┌─────────────────────────────────────────────────────────┐
│                    MCP Servers                          │
│              (GitHub, DB, Figma, Notion...)             │
├─────────────────────────────────────────────────────────┤
│  CLAUDE.md  │  Skills  │  Subagents  │  Hooks          │
│  (Memory)   │  (Reuse) │  (Parallel) │  (Auto)         │
├─────────────────────────────────────────────────────────┤
│              Plan Mode + Context Management             │
└─────────────────────────────────────────────────────────┘
```

| Tool | Use when | Example |
|---|---|---|
| **CLAUDE.md** | Claude needs to remember context every session | Stack, conventions, commands |
| **Skills** | Repeated workflows | `/new-feature`, `/deploy`, `/review` |
| **Plugins** | Need a common workflow installed quickly | PR review, frontend design |
| **Subagents** | Task needs specialist expertise or parallelism | Security review, test generation |
| **Hooks** | Actions that must happen every time, no exceptions | Lint after edit, notify when done |
| **Plan Mode** | Complex task, multiple files, unclear direction | Large refactor, new feature |
| **MCP** | Connect to external services | GitHub PRs, database queries |
| **Git Worktrees** | Multiple AI tasks running in parallel | Feature A + Feature B simultaneously |

---

## 3. CLAUDE.md — The Project's Brain

### Auto-loading mechanism

CLAUDE.md is loaded **automatically** every session. Hierarchy from global → local:

```
~/.claude/CLAUDE.md              ← Global (applies to all projects)
~/.claude/rules/                 ← Global rules split into topic files, all auto-loaded
    ├── code-style.md            ← Each file = 1 topic, loaded alongside CLAUDE.md
    ├── security.md
    └── workflow.md
project/CLAUDE.md                ← Project-wide (commit to git)
project/CLAUDE.local.md          ← Personal (gitignore, never commit)
project/front/CLAUDE.md          ← Loaded when working in front/
project/back/CLAUDE.md           ← Loaded when working in back/
```

**When to use `~/.claude/rules/`?**

Instead of cramming everything into one long `~/.claude/CLAUDE.md` → split into small files by topic. Claude loads all files in `rules/` automatically, no registration needed.

```
# Example rules/ organization
~/.claude/rules/
    ├── code-style.md     ← Formatting, naming conventions across all projects
    ├── security.md       ← Mandatory security rules
    ├── git.md            ← Commit, branch, PR conventions
    └── no-do.md          ← Things Claude must never do (override defaults)
```

> When you work with `front/src/components/`, Claude automatically loads: Global + rules/ → Project → `front/` → `front/src/components/` (if exists).

### Principles for a good CLAUDE.md

**Keep it short — under 150 lines for root CLAUDE.md**

Claude Code's system prompt already has ~50 built-in instructions. A too-long CLAUDE.md causes Claude to start ignoring rules in the middle.

```markdown
# ✅ INCLUDE — what Claude can't know on its own
- Bash commands to run the project
- Code style differences from default (tabs vs spaces, line width 120)
- Team branch naming conventions
- Project-specific gotchas
- Required env vars to run locally

# ❌ EXCLUDE — Claude already knows or doesn't need
- "Write clean code"
- Standard language conventions
- Frequently changing information
- File-by-file codebase descriptions
- Long explanations / tutorials
```

> **Tips to increase adherence:**
> - Use `IMPORTANT` and `YOU MUST` for absolute rules
> - Test each rule: if Claude already does it right without the rule → delete it
> - Use `@import` to modularize instead of cramming everything into one file
> - Commit `CLAUDE.md` to git so the whole team benefits


### Basic template

````markdown
# CLAUDE.md

## Project
[1-2 sentences describing what the app is]

## Stack
[Brief tech stack]

## Commands
```bash
npm run dev      # Start dev server
npm test         # Run tests
npm run lint     # Lint code
```

## Conventions
- [Only what differs from defaults]

## YOU MUST
- [Rules that must never be broken]
````

See: [templates/CLAUDE.md.example](./templates/CLAUDE.md.example)

---

## 4. Skills — Reusable Workflows

### What are Skills

Skills = workflows packaged into `.claude/skills/<name>/SKILL.md` files. Each skill automatically gets a slash command `/name`.

> As of 2026, skills and commands have been merged — `.claude/commands/` still works but `.claude/skills/` is the standard.

### Key frontmatter

```yaml
---
name: fix-bug                    # → slash command /fix-bug
description: |                   # Claude uses this for auto-invoke (semantic matching)
  Fix bugs in the codebase.
  Use when: user reports a bug, test fails, error in logs
disable-model-invocation: true   # Claude won't self-invoke (for deploy, commit)
user-invocable: false            # Hide from / menu (background knowledge)
allowed-tools: Bash(pytest *), Read, Grep
context: fork                    # Run in separate subagent
agent: Explore                   # Subagent type when context: fork
model: sonnet                    # Override model for this skill
argument-hint: "<issue-id>"      # Autocomplete hint
---
```

### 3 types of skills

**Type 1: Auto-invoked** — Claude calls it automatically when the task matches

```yaml
---
name: api-conventions
description: REST API conventions for this project. Use when creating or editing an API endpoint.
user-invocable: false
---
# API Conventions
- Use kebab-case for URLs
- camelCase for JSON fields
- Every endpoint must have an operation_id
```

**Type 2: User-invoked** — you call it manually

```yaml
---
name: deploy
description: Deploy to production
disable-model-invocation: true
allowed-tools: Bash(*)
---
Deploy $ARGUMENTS:
1. Run full test suite
2. Build Docker image
3. Push and deploy to cluster
4. Health check — rollback if failed
```

**Type 3: Subagent skill** — runs in its own context, doesn't pollute main context

```yaml
---
name: security-review
description: Review code for security issues
context: fork
agent: Explore
allowed-tools: Read, Grep, Glob
---
Review $ARGUMENTS for security vulnerabilities.
Report: file:line, severity (Critical/High/Medium/Low), suggested fix.
```

### Best practices

- Description must contain keywords users will say → Claude uses semantic matching to auto-invoke
- Keep SKILL.md under 500 lines; detailed docs go in separate files with `@reference`
- `$ARGUMENTS` for all args, `$0 $1 $2` for positional args
- Whitelist specific commands: `allowed-tools: Bash(pytest *)` instead of all `Bash(*)`

See: [templates/skill.md.example](./templates/skill.md.example) · [everything-claude-code/.claude/](https://github.com/affaan-m/everything-claude-code/tree/main/.claude)

---

## 5. Plugins — Pre-packaged Skills

> Plugin = bundle of skills + hooks + agents + MCP servers. Install once, get everything.

### Install plugins

```bash
# Browse marketplace
/plugin

# Install specific plugin
/plugin install frontend-design@claude-plugins-official
/plugin install pr-review-toolkit@claude-plugins-official
/plugin install feature-dev@claude-plugins-official
```

### Featured plugins

| Plugin | Use when |
|---|---|
| **frontend-design** | Build UI — auto-invoked for frontend work, avoids "AI slop" aesthetics |
| **feature-dev** | Guided feature workflow with 3 agents: code-explorer, code-architect, code-reviewer |
| **pr-review-toolkit** | Comprehensive PR review: comments, tests, error handling, types |
| **code-review** | 5 parallel Sonnet agents: CLAUDE.md compliance, bug detection, PR history |
| **aws-deploy** | Deploy to AWS with architecture recommendations |
| **database** | Schema design, migrations, query optimization |

### Plugin vs custom Skill

```
Use Plugin when:                  Write custom Skill when:
✅ Common workflow                 ✅ Project-specific conventions
✅ Need it now, no setup wanted   ✅ Integration with internal tools
✅ Available in marketplace        ✅ Specific business logic
                                   ✅ No suitable plugin exists
```

---

## 6. Subagents — Delegation & Parallelism

### Why you need Subagents

**Context poisoning:** When Claude researches/explores the codebase, it reads hundreds of files → context window fills with irrelevant information → performance degrades significantly.

Subagents run in their **own separate context window**. The main conversation stays clean.

### Built-in subagents (no config needed)

| Agent | Model | Tools | Use for |
|---|---|---|---|
| **Explore** | Haiku (fast) | Read-only | Research, investigation |
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

You are a security engineer specializing in OWASP.
Review for: XSS, SQL injection, CSRF, auth bypass, data exposure.
Format: Critical/High/Medium/Low with file:line and suggested fix.
```

Place at: `.claude/agents/security-auditor.md`

### Worktree Isolation (2026)

```yaml
isolation: worktree   # Subagent works on its own git worktree
```

Multiple subagents can edit files in parallel without conflicts. Worktrees are automatically cleaned up if there are no changes.

### Effective patterns

**Writer/Reviewer pattern — clean context, unbiased review:**

```
Session A: Implement rate limiter
Session B: Review rate limiter implementation
```

**Parallel analysis:**

```
"Use subagents in parallel:
 - Subagent 1: Security review of auth module
 - Subagent 2: Generate tests for auth module
 Report results back to main session."
```

**Investigation before implementation:**

```
"Use an Explore subagent to research the codebase,
 find existing patterns in back/app/services/.
 Just summarize — don't pollute the main context."
```

---

## 7. Hooks — Deterministic Automation

### Hooks vs CLAUDE.md rules

| | CLAUDE.md rules | Hooks |
|---|---|---|
| Nature | Advisory | Deterministic |
| Can be skipped | Yes | No |
| Configured in | CLAUDE.md | `.claude/settings.json` |
| Use when | Convention, style | Must always run |

### Lifecycle events

```
PreToolUse    → Before Claude uses a tool (can block the action)
PostToolUse   → After a tool finishes running
Notification  → When Claude wants to notify the user
Stop          → When Claude finishes a task
```

### Configure in `.claude/settings.json`

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
          "command": "osascript -e 'display notification \"Claude finished\" with title \"Claude Code\"'"
        }]
      }
    ]
  }
}
```

### Real-world use cases

| Hook | Trigger | Command |
|---|---|---|
| Auto lint | After every Edit/Write | `ruff check . --fix` |
| Auto format | After every Edit/Write | `prettier --write` |
| Desktop notify | On Stop | `osascript notification` |
| Log session | On Start | Timestamp script |
| Block dangerous ops | PreToolUse | Check and exit if dangerous |

> **Tip:** Use AI to write hooks: `"Write a hook that runs eslint after every TypeScript file edit."`

---

## 8. Plan Mode — Design Before You Build

### Activate

```bash
claude --permission-mode plan     # Start session in Plan Mode
Shift+Tab (twice)                 # Toggle during a running session
```

In Plan Mode: Claude can only **read** (Read, Grep, Glob) — no edits, no commands with side effects.

### 4-step workflow

```
1. EXPLORE  (Plan Mode)
   Claude reads codebase, understands structure
   → "Read @back/app/ and understand the current layered architecture"

         ↓

2. PLAN  (Plan Mode)
   Claude creates a detailed implementation plan
   → "I want to add OAuth. List all files to change and in what order."
   → Ctrl+G to open and edit the plan in your editor before proceeding

         ↓

3. IMPLEMENT  (Normal Mode)
   Switch to Normal Mode, start executing
   → "Implement according to the plan. Verify after each step."

         ↓

4. COMMIT
   → "Commit with a descriptive message and create a PR"
```

### When to use Plan Mode

```
✅ Task touches 3+ files
✅ Unfamiliar with the part of the codebase being changed
✅ New complex feature
✅ Large refactor

❌ Fix typo, rename variable
❌ Small task that can be described in one sentence
❌ Add a log line
```

### Thinking Mode (Extended Thinking)

```bash
claude --thinking          # Enable extended thinking
/thinking on               # Toggle during session
```

Use for especially complex tasks requiring deep reasoning — not for every task.

---

## 9. Context Management — Your Most Precious Resource

### Why it matters

The context window holds: the entire conversation + every file Claude reads + every command output. When full, Claude starts "forgetting" instructions and making more mistakes.

### Context management commands

```bash
/clear                          # Reset the entire context
/context                        # See % of context window in use
/compact                        # Compress context, keep important parts (when context window 70-80%)
/compact focus on API changes   # Compress with specific instruction
/btw <question>                 # Quick question — NOT added to context, no token cost
Esc                             # Stop Claude immediately, keep context
Esc+Esc  /  /rewind             # Open rewind menu, restore previous checkpoint
```

### Context management rules

```
✅ /clear after each completed feature
✅ /clear when switching task types (backend → frontend)
✅ /clear after fixing the same bug twice and still failing
✅ Use subagents for investigation → saves main context
✅ /rename <name> to name sessions for easy resuming

❌ Don't /clear mid-task when work is incomplete
❌ Don't let one session contain too many different types of tasks
```

### Resume sessions

```bash
claude --continue       # Continue the most recent session
claude --resume         # Choose from list of named sessions
```

### Checkpoints

Claude automatically checkpoints before every change. Use `Esc+Esc` → rewind menu to:
- Restore conversation only
- Restore code only
- Restore both

---

### Session Handoff — Never Lose Context

> Claude has no memory between sessions. New sessions start from zero → wasted time re-orienting, easy to go in the wrong direction.

**Solution:** Use CLAUDE.md as "long-term memory" — update at the end of each session, read at the start of each new one.

```
End session → Update CLAUDE.md
                    ↓
Start new session → Claude reads CLAUDE.md → Continue exactly where you left off
```

**Step 1 — End the session properly**

```
"Before we finish, update CLAUDE.md with:
 - What we completed
 - Current state of each part
 - Next steps
 - Important decisions & reasoning"
```

**Step 2 — Start a new session properly**

```
"Read CLAUDE.md and summarize for me:
 Where the project stands, what the next steps are,
 and anything I should be aware of?"
```

**Step 3 — Plan Mode for big sessions (> 15 min)**

```
"Read CLAUDE.md and summarize the current project state,
 list what we'll do in today's session,
 what to prioritize and in what order?"
```

**Suggested sections in CLAUDE.md to support handoff:**

```markdown
## Current State
<!-- Claude updates at the end of each session -->
- **Last session:** [date] — [what was completed]
- **In progress:** [feature/task in progress]
- **Next:** [specific next step]

## Key Decisions
<!-- Don't delete — append new decisions as they're made -->
- [Decision] — [reasoning] — [date]
```

> **Tip:** Spend 2 minutes at the end of each session updating CLAUDE.md → save 10 minutes at the start of every next session. See: [docs/en/session-handoff.md](./docs/en/session-handoff.md)

---

## 10. MCP — Connecting External Tools

### What is MCP

Model Context Protocol = universal adapter connecting Claude to external services. Each MCP server exposes tools, resources, and prompts as slash commands.

### Installation

```bash
claude mcp add playwright npx @playwright/mcp@latest
claude mcp add github gh auth login
```

### Deferred Tool Loading (2026)

Claude Code doesn't load full tool schemas on start — only loads names, fetches schemas on demand. Significantly reduces context overhead when using 50+ MCP tools.

### Useful MCP servers

| Server | Use for |
|---|---|
| **GitHub MCP** | Create PRs, read issues, review comments |
| **Playwright MCP** | Automated UI testing |
| **Database MCP** | Query DB directly from Claude |
| **Figma MCP** | Read designs, implement from Figma |
| **Notion MCP** | Read/write tasks and documents |


> **Tip:**
> - Install gh CLI for Claude to use GitHub API more effectively
> - Claude can learn new CLIs: Use `foo-cli --help` to learn how to use it, then solve X.
> - Use `/permissions` to allowlist domains Claude can fetch

---

## 11. Git Worktrees — Safe Parallelism

### The problem

Multiple Claude sessions editing the same directory → conflicts. Git worktrees create a **separate working tree copy** for each branch → sessions work independently.

### Basic setup

```bash
# Create worktrees
git worktree add ../my-project-feature-a feat/feature-a
git worktree add ../my-project-feature-b feat/feature-b

# Run Claude Code in each worktree
cd ../my-project-feature-a && claude   # Terminal 1
cd ../my-project-feature-b && claude   # Terminal 2
```

### Subagent worktrees (automatic)

```yaml
# .claude/agents/parallel-worker.md
---
isolation: worktree
---
```

Subagent creates its own worktree, works independently, cleans up when done.

### Real-world pattern

```
Worktree 1: Developing feature-auth
Worktree 2: Reviewing + fixing production bug
Worktree 3: Refactoring old module
→ 3 Claude sessions running in parallel, no conflicts
```

---

## 12. Effective Prompting

### Core principles

**Describe the outcome, don't micromanage how:**

```
❌ "Open file users.py, find class User, add method validate_email..."
✅ "Add email validation to the User model. Run tests when done."
```

**Provide verification criteria:**

```
❌ "Implement email validation"
✅ "Implement email validation. Test cases:
    user@example.com → True
    invalid → False
    user@.com → False
    Run tests and confirm all green."
```

**Reference with `@`:**

```
✅ "Follow the same pattern as @back/app/services/user_service.py"
✅ "Read @SPEC.md before implementing"
✅ "Schema at @back/app/schemas/user.py"
```

**Be specific about scope:**

```
❌ "Fix the login bug"

✅ "User reports login fails after session timeout.
    Check auth flow at src/auth/, especially token refresh.
    Write a failing test that reproduces the issue, then fix it."
```

### Let Claude interview you for large/ambiguous tasks

```
I want to build [brief description].

Interview me in detail using the AskUserQuestion tool.
Ask about: implementation, UX, edge cases, tradeoffs, security.
Don't ask about obvious things — dig into the hard parts I haven't thought about.
Interview until everything is covered, then write the spec to SPEC.md.
```

### Course-correct early

```
Esc          → Stop immediately when you see the wrong direction
Esc+Esc      → Rewind to the nearest good checkpoint

After fixing the same bug twice and still failing:
  → /clear + write a better prompt based on what you've learned
    (don't continue the correction loop)

Vague prompts are OK when exploring:
  → "What would you improve in this file?"
```

---

## 13. Common Anti-patterns

### Kitchen sink session

```
❌ One session: debug bug → add feature → refactor → setup CI/CD
   → Context filled with garbage, performance degrades

✅ Fix: /clear between different types of tasks
```

### Infinite correction loop

```
❌ Claude wrong → fix → still wrong → fix again → context full of failed attempts

✅ Fix: After 2 failures → /clear, write a better prompt based on what you learned
```

### Over-specified CLAUDE.md

```
❌ CLAUDE.md > 150 lines → Claude ignores middle rules

✅ Fix: Prune ruthlessly. Use Skills/Subagents for domain-specific knowledge.
```

### Trust-then-verify gap

```
❌ Claude generates code that looks correct but doesn't handle edge cases → ship immediately

✅ Fix: Always verify (tests, scripts, screenshots).
        No verification = don't ship.
```

### Infinite exploration

```
❌ "Investigate the entire codebase" → Claude reads hundreds of files → context full

✅ Fix: Scope the investigation narrowly or use an Explore subagent
```

### Skipping Plan Mode for complex tasks

```
❌ Start coding immediately without exploring → solving the wrong problem, have to redo

✅ Fix: Use Plan Mode for tasks touching 3+ files or unfamiliar code areas
```

### Merging AI output without review

```
❌ Claude doesn't know business context.
   Output looks syntactically correct but logically wrong.

✅ Fix: Always read the diff before committing. The human gate cannot be skipped.
```

---

## 14. Quick Reference

### Session commands

```bash
# Starting up
claude                              # New session
claude --continue                   # Continue most recent session
claude --resume                     # Choose session to resume
claude --permission-mode plan       # Start in Plan Mode

# During session
/clear                              # Reset context
/context                            # View context usage (%)
/compact                            # Compress context
/compact focus on API changes       # Compress with intent
/btw <question>                     # Quick question, no context cost
/rename <name>                      # Name the session
/agents                             # View/create subagents
/rewind                             # Open checkpoint menu
/thinking on                        # Enable extended thinking

# Keyboard shortcuts
Esc                                 # Stop Claude
Esc+Esc                             # Stop + rewind
Shift+Tab                           # Switch permission mode
Ctrl+G                              # Open plan in editor
```

### Non-interactive (CI/scripts)

```bash
claude -p "prompt"                              # Run once
claude -p "..." --output-format json            # Structured output
claude --permission-mode auto -p "fix lint"     # Auto mode
```

### Plugin commands

```bash
/plugin                             # Browse marketplace
/plugin install <name>@claude-plugins-official
```

### MCP

```bash
claude mcp add <name> <command>
claude mcp list
claude mcp remove <name>
```

---

## Contributing

All contributions welcome! Create an issue or pull request if you have useful tips, real-world patterns, or find outdated information.

---

<div align="center">

Made with ❤️ for developers everywhere

[🇻🇳 Đọc bằng tiếng Việt](./README.vi.md)

</div>
