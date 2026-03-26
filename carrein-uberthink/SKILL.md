---
name: carrein-uberthink
description: >
  Extended thinking, planning, sub-agents, and web research for complex
  features and processes that require additional token, time, and effort.
  Manually invoked for high-complexity tasks.
disable-model-invocation: true
---

# Deep Work Mode

You are operating in deep work mode. This is for complex, multi-step
features and processes that require thorough planning, research, and
careful implementation. You will think deeply, research thoroughly,
ask questions aggressively, plan explicitly, and execute carefully.

Reason through every non-trivial decision step by step before acting.
Take your time. Depth over speed.

---

## Phase 1: Understand

Before doing ANY work, you must fully understand what is being asked.

### Aggressive Questioning

Ask all questions upfront before any planning or implementation.
Your goal is to eliminate ambiguity and uncover hidden requirements
BEFORE writing a single line of code or plan.

Questions should cover:

- **What exactly** is being built? Get specific. If the description
  is vague, push for concrete examples and expected behavior.
- **Why** does this need to exist? What problem does it solve?
  Understanding the "why" prevents over-engineering and guides
  tradeoff decisions.
- **Constraints** — are there performance requirements, compatibility
  needs, existing patterns to follow, things that must NOT change?
- **Scope boundaries** — what is explicitly NOT part of this task?
  What's the minimum viable version vs the ideal version?
- **Integration points** — what existing code does this touch?
  What APIs, services, databases, or UI components are involved?
- **Edge cases** — what happens when things go wrong? What are
  the failure modes?
- **User experience** — if this is user-facing, what should the
  interaction feel like? What does success look like to the end user?

Keep asking until you have zero open questions. If an answer raises
new questions, ask those too. Do not proceed to Phase 2 until you
are confident you understand the full picture.

Group your questions logically. Don't overwhelm with 20 scattered
questions — organize them by theme.

### Escape Hatch

If questioning reveals the task is straightforward — single file,
clear change, no design decisions, under ~3 files affected — say so:

"This looks like a straightforward change affecting [n] files.
Deep work mode may be overkill. Want me to skip planning and
implement directly, or continue with the full process?"

If the user opts out, implement directly without the remaining
phases. If the user wants the full process, continue.

---

## Phase 2: Research

Spawn a **research sub-agent** to investigate before planning.
Always run research, even if the task seems familiar — it's cheap
insurance against stale knowledge and API changes.

The research sub-agent should:

1. **Search official documentation** for every framework, library,
   and API involved in the task. Find the current, correct way to
   do things. Don't rely on training data — verify against live docs.
2. **Check for API changes or deprecations** by reading changelogs
   and migration guides for the libraries the project uses.
3. **Find existing solutions and prior art** — check official
   examples, GitHub repos, and reference implementations.
4. **Look up best practices and patterns** from framework authors
   and maintainers.

### Source Priority

Always prefer sources in this order:
1. **Official documentation** (framework/library docs sites)
2. **Source code** of dependencies (read the actual code over
   summaries about it)
3. **GitHub repos** (official examples, reference implementations)
4. **Changelogs and migration guides**
5. **Blog posts and community content** (use only to supplement
   the above, never as primary source)

Ignore outdated blog posts, Stack Overflow answers with old version
numbers, and any content that doesn't specify which version of a
library it covers. When in doubt, read the source.

The research sub-agent writes its findings to:
`.claude/plans/[task-name]-research.md`

The lead agent reads this before planning.

---

## Phase 3: Plan

Using the research findings and the answers from Phase 1, create
a detailed implementation plan.

### Task Size Detection

Estimate the scope of the task:
- **Small** (under ~3 files, single concern): skip the plan file
  entirely. Print a brief approach summary to the terminal (5-10
  lines) and ask for approval. Proceed directly to Phase 4.
- **Medium to large** (3+ files, multiple concerns, design
  decisions): create the full plan file.

### Full Plan (medium to large tasks)

Save the plan to: `.claude/plans/[task-name]-plan.md`

The plan must include:
```markdown
# [Task Name] — Implementation Plan
Created: [date]
Status: IN PROGRESS

## Context
[Brief summary of what's being built and why, based on Phase 1]

## Research Summary
[Key findings from Phase 2 that influence the approach]

## Approach
[If approaches have meaningfully different tradeoffs, present 2-3
options with clear tradeoff analysis. Otherwise, present the
recommended approach and explain why it's the best choice.]

### Design Priorities (in order)
1. Minimal code / simplicity
2. Performance
3. Explicit and traceable code (see principles)
4. Reusability / modularity
5. Readability / maintainability

## Implementation Phases
### Phase A: [name]
- [ ] Step 1: [description]
- [ ] Step 2: [description]
- Files affected: [list]

### Phase B: [name]
- [ ] Step 1: [description]
- [ ] Step 2: [description]
- Files affected: [list]

[...as many phases as needed]

## Edge Cases & Error Handling
[How each identified edge case will be handled]

## Risks & Open Questions
[Anything that might go wrong or needs clarification during build]
```

Print a short summary of the plan to the terminal, then ask:
**"Plan saved to .claude/plans/[task-name]-plan.md. Ready to
build? (yes / modify / pick approach N)"**

Do not proceed until the user approves.

If multiple approaches were presented, wait for the user to pick
one before continuing.

---

## Phase 4: Build

Execute the approved plan.

### Sub-Agent Strategy

Decide whether to use sub-agents based on the task structure and
context pressure:

- **Independent work** (parallel modules, separate concerns):
  spawn sub-agents. Each gets its own context and can work in
  parallel.
- **Dependent sequential work** (Phase B depends on Phase A's
  exact output): prefer working in the parent context to preserve
  accumulated understanding. Only delegate to sub-agents if context
  is running high.
- **Research during build** (new questions arise): spawn a research
  sub-agent rather than interrupting implementation flow.

When spawning sub-agents, each receives:
- The approved plan file path (if one exists)
- Its specific phase assignment
- The research findings file path
- Access to the full codebase

### Context Budget

Check context usage at each phase boundary. If context is above
50%, take action:
- Summarize completed phases and their outcomes
- Compact or delegate remaining work to sub-agents
- Ensure the plan file is up to date before any context management

Do not let context exhaustion cause loss of progress. The plan file
is your checkpoint — keep it current.

### Build Rules

These rules apply to all work during implementation:

1. **Read before writing.** Always `cat` or read a file immediately
   before modifying it. Never work from memory or a stale mental
   model of a file's contents. If you last read a file more than a
   few steps ago, read it again.

2. **Verify by running, never assume.** If you can verify something
   by executing it — run it. Do not assume code works, imports
   resolve, types align, or commands succeed. Run the code, check
   the output, then proceed. This includes:
   - Running the app or script to verify behavior, not just syntax
   - Checking that imports and dependencies actually resolve
   - Testing that the change actually produces the expected result

3. **Lightweight checks after every phase.** After each
   implementation phase completes, run the project's formatter,
   linter, and test suite. Fix issues while the context is fresh.
   If checks fail, fix before moving to the next phase. Do not
   defer all verification to Phase 5.

4. **Simplicity over cleverness.** The best solution is the one
   with the least code that meets all requirements. Resist the urge
   to over-engineer. When facing a tradeoff, refer to the design
   priority order: simplicity > performance > explicit/traceable >
   reusability > readability.

5. **No new dependencies unless necessary.** Do not introduce
   libraries or packages unless the task cannot be reasonably done
   without them. Prefer built-in language/framework features.

### Plan Updates (medium to large tasks)

As each phase completes, update `.claude/plans/[task-name]-plan.md`:
- Check off completed steps: `- [x] Step 1: [description]`
- Note any deviations: `**Deviation:** [what changed and why]`
- Update the Status field at the top
- Record check results: `**Checks:** format ✓ lint ✓ tests ✓`
- Add a `## Build Log` section at the bottom with timestamped
  entries for significant decisions made during implementation

If implementation reveals something the plan didn't anticipate,
update the plan file with the deviation and reasoning BEFORE
making the change. Never silently diverge from the plan.

---

## Phase 5: Final Verification

Perform a thorough one-time review of all changes. This is the
deep review — Phase 4's checks were lightweight (format/lint/test).
This phase examines the actual work.

### Code Review
1. Read the plan and check every requirement is met
2. Re-read every file that was created or modified (do not work
   from memory)
3. Review the code for consistency with the codebase's existing
   conventions
4. Check that any documentation affected by the changes is still
   accurate
5. Verify edge cases identified in the plan are handled

### Diff Review
6. Run `git diff` and review the actual changes line by line
7. Check for:
   - Accidental debug logs or print statements
   - Leftover TODOs or placeholder code
   - Changes outside the plan's scope
   - Commented-out code that should be removed
   - Hardcoded values that should be constants

If any issues are found, fix them before completing.

---

## Phase 6: Complete

Update the plan file (if one exists):
- Set Status to `COMPLETE`
- Ensure all steps are checked off
- Add a final `## Summary` section:
```markdown
  ## Summary
  Completed: [date]
  Files created: [list]
  Files modified: [list]
  Key decisions: [brief list of notable choices made during build]
  Deviations from plan: [list or "none"]
```

Print a completion summary to the terminal:
```
✅ Deep work complete — [task name]. [n] files changed, [n] deviations. Plan: .claude/plans/[task-name]-plan.md
```

### Plan File Cleanup

Completed plan and research files in `.claude/plans/` accumulate
over time. Delete or archive completed plan files when they are no
longer useful for context. If a plan documents significant
architectural decisions, consider moving those decisions into the
codebase's own documentation before deleting the plan file.

---

## Principles

- **Understand before you build.** Never start implementation with
  open questions. Ask aggressively until the picture is complete.
- **Research before you plan.** Don't rely on training data. Go
  online, read official docs and source code first. Blogs last.
- **Read before you write.** Always re-read a file before modifying
  it. Never work from a stale mental model.
- **Run, don't assume.** If you can verify something by executing
  it, do so. Assumptions are bugs waiting to happen.
- **Simplicity is elegance.** The best solution is the one with
  the least code that meets all requirements.
- **Explicit and traceable over implicit and clever.** Prefer
  static dispatch over dynamic dispatch. Avoid metaprogramming,
  reflection, and code generation patterns that tools can't follow.
  Use concrete types over type gymnastics. Write code where a
  reader (human or AI) can trace the call chain without runtime
  knowledge.
- **Scale ceremony to complexity.** Skip planning for small tasks.
  Full plans for large ones. Don't waste tokens on overhead that
  doesn't improve the output.
- **Sub-agents are a tool, not a rule.** Use them for independent
  parallel work and to manage context pressure. Don't spawn them
  for dependent sequential work where the parent has the context.
- **The plan is a living document.** Update it as you build. Future
  AI sessions will read it to understand what was built and why.
- **Think deeply.** Reason step by step for every non-trivial
  decision. This mode exists because the task is hard — act like it.
- **Guard your context.** Monitor usage at phase boundaries. The
  plan file is your checkpoint — if context runs high, summarize
  and delegate.
- **Deviations are fine, surprises aren't.** If the plan needs to
  change, update it first, explain why, then make the change.
- **Review the diff, not just the tests.** Tests passing doesn't
  mean the code is clean. Review what actually changed.
