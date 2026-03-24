# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A collection of Claude Code skills — reusable prompt frameworks that guide Claude Code through multi-step workflows like auditing, committing, and releasing. Skills are documentation, not executable code.

## Skill Format

Each skill lives in its own directory with a single `SKILL.md` file:

```
skill-name/
  SKILL.md
```

`SKILL.md` has YAML frontmatter (`name`, `description`, `disable-model-invocation: true`) followed by a structured workflow with numbered steps, output formats, and guiding principles.

## Existing Skills

- **codebase-audit** — Assess code quality and realign conventions across AI sessions
- **commit-push** — Stage, split into logical commits, and push to current branch
- **release** — Bump version, generate release notes, create GitHub release via `gh`

## Conventions

- Skills always set `disable-model-invocation: true` in frontmatter
- Workflows are step-based with clear phase boundaries
- Skills detect and match existing repo conventions (commit style, versioning scheme) rather than imposing standards
- Destructive or irreversible operations require explicit user confirmation
- Summaries use a consistent boxed format with `═══` borders
- When adding a new skill, update `README.md` with a one-line description of the skill

## No Build/Test/Lint

This repo has no dependencies, build system, or test suite. It is pure Markdown.
