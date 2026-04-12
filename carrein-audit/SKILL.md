---
name: carrein-audit
description: >
  Assess code quality and realign conventions that have drifted across
  AI sessions. Use when the user says "audit", "code quality",
  "convention check", "codebase audit", or "align conventions".
disable-model-invocation: true
---

# Codebase Audit

You are performing a code quality audit of a codebase. This is a
long-running, thorough operation focused on a single concern:

**Code quality** — detect drift in conventions, patterns, and standards
across AI sessions, then realign the codebase with its own dominant
conventions.

This skill is invoked manually when the user feels things may have
drifted. There is no fixed schedule — treat every invocation as
"it's been a while, check everything."

---

## Phase 0: Scope & Orientation

### Detect Context

- Identify the primary language(s) and framework(s) in use
- Read existing config files (linters, formatters, tsconfig, pubspec,
  pyproject, etc.) to understand the project's declared standards
- Scan folder structure to understand the architecture
- Identify the dominant conventions already in use — these are the
  baseline to align toward, not an external standard

### Gather the Full Picture

This is a long-running skill. Do not shortcut the research phase.
Build a complete understanding before assessing anything:

- **Current state:** `git status`, `git diff` — what's in progress
  right now
- **Recent history:** `git log --oneline -30` and `git diff` across
  recent commits to understand what has changed since the codebase
  was last "known good"
- **Releases:** check tags (`git tag --sort=-creatordate | head -5`)
  to understand the release cadence and what shipped recently
- **Previous audit breadcrumbs:** check for `.claude/audit-marker`
  — if it exists, read `last_audit_commit` and diff from that point.
  If it doesn't exist, audit the full current state.

### Exclusions

Always skip the following. Do not read, assess, or modify these:

- Dependency directories: node_modules/, .dart_tool/, __pycache__/,
  venv/, .venv/, vendor/
- Build output: build/, dist/, .next/, out/, .build/
- Generated code: *.g.dart, *.freezed.dart, *.gen.ts, *.generated.*,
  and any file with a "DO NOT EDIT" or "GENERATED" header
- Lock files: package-lock.json, pubspec.lock, yarn.lock,
  pnpm-lock.yaml, poetry.lock, Gemfile.lock, go.sum
- Assets: images, fonts, icons, SVGs, audio, video files
- Anything matched by the project's .gitignore
- **README.md** — the user maintains this manually

When in doubt whether a file is generated, check for a generated
header comment in the first 5 lines. If present, skip it.

### Size Assessment & Agent Strategy

- Count total source files and lines (after exclusions)
- If the repo has **fewer than 500 source files**: proceed as a
  single agent
- If the repo has **500+ source files**: use multi-agent strategy:
  1. Identify logical modules (top-level directories, packages, or
     workspace members)
  2. Spawn one sub-agent per module — each runs Phase 1
     independently against its module
  3. This lead agent collects all sub-agent findings, deduplicates
     cross-module issues, and produces the unified report
  4. Cross-cutting concerns (shared utilities used across modules,
     inconsistent patterns BETWEEN modules) are assessed by the lead
     agent after sub-agents complete

Print the scope summary before starting:

```
Scope: [n] source files, [n] lines across [n] modules
Strategy: [single agent | multi-agent with N sub-agents]
Modules: [list of module names]
Last audit: [commit ref from breadcrumb, or "none found"]
Starting audit...
```

---

## Phase 1: Assess

Evaluate source code across six categories, in this priority order:

### 1. Performance (highest priority)

- Unnecessary re-renders, redundant computations, N+1 patterns
- Synchronous operations that should be async
- Large imports where tree-shakeable or lighter alternatives exist
- Missing caching, memoization, or indexing where obviously
  beneficial
- Resource leaks (unclosed streams, connections, listeners)

### 2. Error Handling

- Swallowed errors (empty catch blocks, ignored futures)
- Inconsistent error handling strategies across the codebase
- Missing error boundaries or fallback behavior
- Generic error messages that lose context
- Unhandled edge cases (null, empty, malformed input)

### 3. Test Coverage

- Critical paths without test coverage
- Tests that exist but are stale (testing old behavior)
- Inconsistent testing patterns across similar modules
- Missing edge case tests for error paths

### 4. Type Safety

- Weak or missing types (any, dynamic, Object, untyped maps)
- Inconsistent type usage across similar functions
- Type assertions or casts that bypass safety
- Missing null safety or optional handling

### 5. Modularity & Reuse

- Duplicated logic that should be extracted into shared utilities
- Large files doing too many things (separation of concerns)
- Functions with multiple responsibilities
- Tightly coupled modules that should be independent
- Logic mixed into UI or data layers where it doesn't belong

### 6. Readability & Naming

- Inconsistent naming conventions (camelCase in some files,
  snake_case in others for the same language)
- Ambiguous or misleading function/variable names
- Inconsistent file/folder naming patterns
- Dead code, unused imports, commented-out blocks

---

## Phase 2: Documentation Drift Scan

This is a lightweight scan to detect whether documentation has drifted.
Do NOT fix anything in this phase — flag only.

### What to Check

- **All CLAUDE.md files** (root + nested) — do listed items still
  exist? Are conventions still followed?
- **Documentation directory** (docs/, doc/, or similar) — any
  obviously stale content?

### What NOT to Check

- Do NOT check memory files (that is the sync skill's job)
- Do NOT check README.md (user maintains manually)

### Output

```
Documentation drift: [none detected | minor drift | significant drift]
[If drift found: 1-3 line summary of what drifted]
[If drift found: "Run /sync to bring documentation into parity."]
```

---

## Phase 3: Report & Approval

This is the critical gate. Present everything, ask questions, wait
for approval. Do NOT make any changes until the user says go.

### Part A: Code Quality Report

```
══════════════════════════════════════════════════
  CODE QUALITY REPORT — [project name]
  Language: [detected]  |  Framework: [detected]
  Files scanned: [n]    |  Lines: [n]
  Strategy: [single | multi-agent, N modules]
══════════════════════════════════════════════════

  1. [R/Y/G] Performance          [one-line summary]
  2. [R/Y/G] Error Handling       [one-line summary]
  3. [R/Y/G] Test Coverage        [one-line summary]
  4. [R/Y/G] Type Safety          [one-line summary]
  5. [R/Y/G] Modularity & Reuse   [one-line summary]
  6. [R/Y/G] Readability & Naming [one-line summary]

  Documentation drift: [none | minor | significant]

══════════════════════════════════════════════════
```

Traffic light criteria:
- Green: Consistent across the codebase, no significant issues
- Yellow: Some drift or minor issues found, worth addressing
- Red: Significant inconsistency or problems, should fix

### Part B: Findings by Severity

List all findings grouped by severity (Critical / Moderate / Minor),
each with:
- **File(s):** [paths]
- **Category:** [which of the 6]
- **Issue:** [what's wrong]
- **Proposed fix:** [what the change would be, briefly]

Then ask:
**"Which groups should I fix? (e.g., 'all', 'all red', 'red and yellow', or 'none')"**

Wait for explicit approval. Do not proceed without it.

---

## Phase 4: Apply Approved Fixes

### Code Quality Fixes

1. Make changes that align the codebase with its own dominant
   conventions
2. After each code change, check if any inline comments or
   docstrings near the changed code are now inaccurate — if so,
   update them
3. Run any available linter, formatter, or test suite to verify
   nothing is broken
4. If a change breaks tests or the build, revert that specific
   change, report it as unresolvable, and continue with the rest

### Leave a Breadcrumb

After all changes are applied, write to `.claude/audit-marker`:

```
last_audit_commit: [current HEAD commit hash]
last_audit_date: [today's date]
last_audit_files_checked: [count]
last_audit_changes: [brief summary]
```

If the file already exists and contains `last_sync_*` fields,
preserve those fields.

Do not commit. The user will commit when ready.

### Print Summary

```
══════════════════════════════════════════════════
  AUDIT COMPLETE
══════════════════════════════════════════════════

  Fixes applied: [n]
  Documentation drift: [none | minor — run /sync | significant — run /sync]

  Breadcrumb written to .claude/audit-marker
  No commit created. Run /commit when ready.
══════════════════════════════════════════════════
```

---

## Phase 5: Context Check & Compact Recommendation

After all work is done, check the current context usage.

If context is **above 50%**, recommend compacting:

```
══════════════════════════════════════════════════
  CONTEXT CHECK
══════════════════════════════════════════════════

  Context usage: [estimated %]

  Recommendation: COMPACT & NEW CONVERSATION
  This audit consumed significant context. Starting a
  fresh conversation will give the next task a clean
  context window with all updated docs and memory
  loaded automatically.

  Compact now? (yes / no)
══════════════════════════════════════════════════
```

If the user says yes, summarize the key outcomes of this audit
session into a compact message (what was changed, what decisions
were made, any open items) and then compact the conversation.

If context is **below 50%**, simply note it:

```
  Context usage: [estimated %] — plenty of room.
```

---

## Principles

- **Align with the codebase, not with you.** The most common existing
  pattern is the correct one. Do not introduce new conventions.
- **Convention convergence over preference.** If 80% of files use one
  pattern, align the other 20% to match.
- **AI-readability is a first-class goal.** Every change should make
  it easier for the next AI session to understand the project.
- **Never change behavior.** Code quality fixes are alignment, not
  feature work. Inputs and outputs of every function must stay the
  same.
- **Questions before actions.** When something is ambiguous, ask.
  Never silently resolve ambiguity with a guess.
- **Scan everything, touch only what's drifted.** Read every file in
  scope. Edit only what's wrong.
- **Language-agnostic assessment.** Apply the standards and idioms
  appropriate to whatever language and framework the project uses.
- **Do not skip files to save time.** Thoroughness is more important
  than speed. This is a long-running skill by design.
- **Leave the trail.** The breadcrumb file exists so the next audit
  can be smarter about what changed since last time.
