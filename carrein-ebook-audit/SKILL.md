---
name: carrein-ebook-audit
description: >
  Audit an ebook library — check filenames, covers, integrity, and flag
  issues. Use when the user says "audit library", "check ebooks",
  "audit ebooks", "library audit", or "check my books".
---

# Ebook Library Audit

You are auditing an ebook collection in the current folder. Your job is
to read the audit checklist defined in the local CLAUDE.md, execute each
step in order, verify filenames, covers, and file integrity, and present
all proposed changes for user approval before touching anything.

---

## Step 1: Read the Audit Checklist

- Read `CLAUDE.md` in the current directory (or `.claude/CLAUDE.md`)
  for the full audit checklist (steps 1-12)
- If no `CLAUDE.md` exists or it has no audit steps, stop and tell the
  user — do not improvise a checklist

---

## Step 2: Execute Audit Steps 1-10

- Run each audit step (1 through 10) in the order defined in CLAUDE.md
- Record every finding: bad filenames, missing covers, corrupt files,
  duplicate formats, metadata issues, and anything else the checklist
  calls for
- Use online sources (Goodreads, Google Books) to verify correct titles
  and author spellings when the checklist requires it
- Prefer EPUB over PDF when both formats exist for the same book

---

## Step 3: Cover Upgrade (Step 11)

- Run the cover upgrade script:
  python3 cover_upgrade.py .
- The script and its cache (`.cover_cache.json`) live in the stories
  folder alongside the ebooks
- The cache skips files whose content hasn't changed since last audit
- If the user asks for a fresh or full re-audit, pass `--no-cache`:
  python3 cover_upgrade.py . --no-cache
- Review the report output
- If upgrades are available, present them to the user and ask for
  confirmation before running with `--apply`
- If no upgrades are found, note this and move on

---

## Step 4: Collect & Confirm Changes (Step 12)

Collect ALL proposed changes from every audit step — renames, deletions,
flags, cover upgrades — into a single summary table:

══════════════════════════════════════════════════
  EBOOK LIBRARY AUDIT — [folder name]
  Files scanned: [n]
  Issues found:  [n]
══════════════════════════════════════════════════

  Renames:
1. [old filename] -> [new filename]    [reason]
2. ...

  Deletions:
1. [filename]                          [reason]
2. ...

  Flags (manual review needed):
1. [filename]                          [issue]
2. ...

  Cover Upgrades:
1. [filename]                          [old -> new source]
2. ...

══════════════════════════════════════════════════

Wait for explicit user confirmation before executing any changes.
Do not apply partial changes — present everything, then act on what
the user approves.

---

## Principles

- **Never modify without approval.** All renames, deletions, and file
  changes require explicit user confirmation. Present first, act second.
- **Follow the local checklist.** The CLAUDE.md in the ebook directory
  is the source of truth for audit steps. Do not add or skip steps.
- **EPUB over PDF.** When duplicate formats exist, prefer EPUB.
- **Verify with external sources.** Use Goodreads, Google Books, or
  other reliable sources to confirm correct titles and author spellings.
- **One summary, one confirmation.** Batch all proposed changes into
  a single table so the user can review everything at once.
