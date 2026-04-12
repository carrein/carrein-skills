---
name: carrein-sync
description: >
  Sync project documentation, CLAUDE.md files, and memory with the
  current codebase state. Use when the user says "sync", "sync docs",
  "update context", "sync claude", "update claude.md", or "refresh docs".
disable-model-invocation: true
---

# Documentation & Context Sync

You are syncing project documentation and context with the current
state of the codebase. Your job is to detect drift between what the
docs/memory say and what the code actually does, then bring everything
into parity.

This is a focused, moderate-weight operation. It does NOT assess code
quality — run /audit for that. It checks whether documentation and
memory accurately reflect the codebase as it exists right now.

---

## Phase 0: Orientation

Lightweight setup — no language/framework detection needed (that's audit's job). Just gather what's needed for doc comparison.

- **Git state:** `git status`, `git diff`, `git log --oneline -20`
- **Recent tags:** `git tag --sort=-creatordate | head -5`
- **Breadcrumb:** Read `.claude/audit-marker` if it exists. Use `last_sync_commit` to determine what changed since last sync. If no marker, sync against full current state.
- **Find all documentation files:**
  - All CLAUDE.md files (root + nested per-directory ones)
  - Documentation directory — scan for `docs/`, `doc/`, `documentation/`, or any directory containing primarily `.md` files. Don't assume the name.
  - ARCHITECTURE.md, CONTRIBUTING.md, or any `.md` files referenced from CLAUDE.md
  - Do NOT check README.md — user maintains manually
- **Find memory files:** Read the memory index (the project's `MEMORY.md`) and all referenced memory files
- Read every documentation and memory file found

Print orientation summary:

```
Scope: [n] documentation files, [n] memory files
Last sync: [commit ref from breadcrumb, or "none — full sync"]
Delta: [n commits / n files changed since last sync, or "N/A"]
Starting sync...
```

---

## Phase 1: Compare Documentation & Context Against Reality

For each file found, compare its claims against the actual codebase.

**CLAUDE.md Files (all of them):**
- Listed items that no longer exist (skills, modules, features, patterns removed or renamed)
- Missing entries for new things that exist in codebase but aren't documented
- Stale conventions that the codebase no longer follows, or new conventions not yet documented
- Wrong file paths or directory structures
- Outdated dependencies or tooling references
- Incorrect descriptions
- Line count check — flag if any CLAUDE.md exceeds 200 lines

**Documentation Directory:**
- Stale documents describing removed features/architecture
- Missing documents for significant new features (flag only — don't create without approval)
- Outdated content that has drifted from code reality
- Broken internal references (links to renamed/deleted files)

**Memory Files:**
- Stale facts referencing files, functions, patterns that no longer exist
- Outdated preferences that contradict current practice
- Missing context — significant project changes not reflected in memories
- Redundant memories — duplicates or things already covered by CLAUDE.md

**Other Docs (ARCHITECTURE.md, CONTRIBUTING.md, etc.):**
- Same checks as documentation directory
- Special attention to setup instructions, architecture descriptions, workflow descriptions

---

## Phase 2: Report, Questions & Approval

This is the gate. Present everything, ask questions, wait for go-ahead.

**Part A: Sync Report**

```
══════════════════════════════════════════════════
  DOCUMENTATION & CONTEXT SYNC REPORT
  Last sync: [commit ref or "none"]
  Delta: [n commits / n files changed since last sync]
══════════════════════════════════════════════════
```

List every proposed change grouped by action:

**CREATE** — new documentation that should exist:
- [file path] — [why, what it should contain]

**UPDATE** — existing docs/memory that have drifted:
- [file path] — [what's wrong, what the fix is]

**DELETE** — docs or memory that are stale:
- [file path] — [why it's stale, what it references that no longer exists]

**NO CHANGE** — checked and still accurate:
- [file path] — up to date

**Part B: Questions**

Aggressively ask questions about anything:
- Out of parity (doc says X, code says Y, unclear which is "right")
- Confusing (contradictory patterns, unclear intent)
- Needs discussion (multiple valid approaches, missing context)

Be specific — reference exact file paths and line numbers.

Then ask: **"Which changes should I apply? (e.g., 'all', 'all except [items]', or 'none')"**

Wait for explicit approval.

---

## Phase 3: Apply Approved Changes

For each approved change:

1. Read each file immediately before editing (never work from memory)
2. **UPDATES:** minimal edit to bring doc in line with reality. Preserve existing tone, formatting, structure.
3. **CREATES:** write new docs matching style and depth of existing docs. Keep minimal.
4. **DELETES:** delete the file. If memory file, also remove its entry from MEMORY.md.
5. **Memory updates:** update content AND frontmatter (name, description). Update MEMORY.md index.
6. **CLAUDE.md budget:** after any CLAUDE.md edit, verify it stays under 200 lines. If it exceeds, flag to user.

Write breadcrumb to `.claude/audit-marker`:

```
last_sync_commit: [current HEAD hash]
last_sync_date: [today's date]
last_sync_files_checked: [count]
last_sync_changes: [brief summary]
```

(Preserve any existing `last_audit_*` fields in the file)

Do NOT commit.

Print summary:

```
══════════════════════════════════════════════════
  SYNC COMPLETE
══════════════════════════════════════════════════

  Created: [list or "none"]
  Updated: [list or "none"]
  Deleted: [list or "none"]
  Memory updated: [list or "none"]

  Breadcrumb written to .claude/audit-marker
  No commit created. Run /commit when ready.
══════════════════════════════════════════════════
```

---

## Phase 4: Context Check & Compact Recommendation

If context above 50%:

```
══════════════════════════════════════════════════
  CONTEXT CHECK
══════════════════════════════════════════════════

  Context usage: [estimated %]

  Recommendation: COMPACT & NEW CONVERSATION
  This sync consumed significant context. Starting a
  fresh conversation will give the next task a clean
  context window with all updated docs and memory
  loaded automatically.

  Compact now? (yes / no)
══════════════════════════════════════════════════
```

If user says yes, summarize outcomes and compact.

If below 50%: `Context usage: [%] — plenty of room.`

---

## Principles

- **Sync, don't invent** — update existing docs to match reality, don't add aspirational content
- **Minimal edits** — change only what has drifted, leave everything else untouched
- **Report-first, changes-second** — the approval gate is sacred
- **Read before write** — always re-read a file immediately before editing
- **Match the document's voice** — terse docs stay terse, detailed docs stay detailed
- **CLAUDE.md budget** — never let a CLAUDE.md exceed 200 lines
- **Memory is part of the picture** — stale memories mislead future sessions
- **Scan everything** — thoroughness over speed
- **When unsure, ask** — flag ambiguity in Phase 2 questions, don't guess
- **Leave the trail** — breadcrumb enables smarter future syncs
- **Stay in your lane** — do not flag or fix code quality issues (suggest /audit instead)
