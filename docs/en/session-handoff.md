# Session Handoff — Never Lose Context

> A workflow for maintaining continuous context between sessions — never lose progress.

---

## The Problem

Every time a Claude session ends, all context is cleared. New sessions start from zero.

**Real consequences:**
- Wasted time re-orienting and re-explaining the project every session
- Easy to go in the wrong direction because Claude lacks context
- Technical decisions are "forgotten" and have to be made again
- Unclear progress — which tasks are done, which aren't

**Solution:** Use CLAUDE.md as "long-term memory" — update at the end of each session, read at the start of each new one.

---

## The Session Handoff Loop

```
End session → Update CLAUDE.md
                    ↓
Start new session → Claude reads CLAUDE.md → Continue exactly where you left off
```

---

## Step 1 — End Sessions Properly

**Before closing the terminal, ask Claude to update CLAUDE.md:**

```
"Before we finish, update CLAUDE.md with:
 - What we completed today
 - Current state of everything in progress
 - Next steps to take
 - Important technical decisions & reasoning"
```

**Or more concisely:**

```
"Update CLAUDE.md: today's progress, next steps, decisions made."
```

**Suggested sections in CLAUDE.md:**

```markdown
## Current State
<!-- Claude updates at the end of each session -->
- **Last session:** [date] — [what was completed]
- **In progress:** [feature/task in progress]
- **Next:** [specific next step]

## Key Decisions
- [Decision 1] — [reasoning]
- [Decision 2] — [reasoning]
```

---

## Step 2 — Start New Sessions Properly

**Instead of jumping into tasks, let Claude read context first:**

```
"Read CLAUDE.md and summarize for me:
 Where the project stands,
 What the next steps are,
 and anything I should be aware of?"
```

**Combined with Plans (if using):**

```
"Read CLAUDE.md and @plans/feat-auth-oauth.md.
 Summarize the current state, then continue with the next task."
```

---

## Step 3 — Plan Mode for Big Sessions

**When the session is expected to take > 15 minutes → use Plan Mode to plan first:**

```bash
# Start in Plan Mode
claude --permission-mode plan

# Or toggle during session
Shift+Tab (twice)
```

**Prompt to plan the session:**

```
"Read CLAUDE.md and summarize the current project state.
 List what we'll do in today's session.
 What to prioritize and in what order?"
```

**After planning, switch to Normal Mode to implement:**

```bash
Shift+Tab   # Toggle back to Normal Mode
```

---

## Full CLAUDE.md Template with Session Handoff

````markdown
# CLAUDE.md

## Project
[Brief description]

## Stack
[Tech stack]

## Commands
```bash
[important commands]
```

## Architecture
[Architectural decisions to remember]

## YOU MUST
[Non-negotiable rules]

---

## Current State
<!-- Update at the end of each session -->
- **Last updated:** [date]
- **Last session:** [what was done]
- **In progress:** [what's being worked on]
- **Next steps:**
  1. [step 1]
  2. [step 2]

## Key Decisions
<!-- Don't delete — append new decisions as they're made -->
- [Decision] — [reasoning] — [date]
````

---

## Advanced Tips

**Use `/rename` for easy resuming:**

```bash
/rename oauth-feature-session-3
```

**Resume the right session:**

```bash
claude --resume   # Choose from list
```

**Combine with Plans/ for more detail:**

Plans/ is for task-level tracking (checking off [x] individual tasks).
CLAUDE.md is for session-level context (overall current state).

Use both for larger projects.

---

> **Takeaway:** Spend 2 minutes at the end of each session updating CLAUDE.md. Save 10 minutes at the start of every subsequent session.
