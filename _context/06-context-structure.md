# Step 6: Context Folder Structure

Set up the `_context` folder conventions for LLM-friendly project documentation.

## Purpose

The `_context` folder serves as:
- Project memory for AI assistants working on the codebase
- Living documentation that evolves with the project
- A place for plans, decisions, and reference materials

## 6.1 Core structure

```
_context/
  00-project.md           # What this project is, goals, audience
  01-initial-setup.md     # Setup instructions (this series)
  02-next-interface-setup.md
  03-ink-cli-setup.md
  04-py-server-setup.md
  05-scripts-and-dev.md
  06-context-structure.md

  plans/                  # Implementation plans, feature specs
  readings/               # Notes on readings, research
  decisions/              # Architecture Decision Records (ADRs)
  dev/                    # API docs, SDK references
  manuals/                # Hardware/protocol documentation
```

## 6.2 Create 00-project.md

Create `_context/00-project.md`:

```markdown
# Project: [Your Project Name]

## Overview

[One paragraph describing what this project is]

## Goals

- [Goal 1]
- [Goal 2]
- [Goal 3]

## Audience

Who is this for? Who will use it?

## Architecture

### Apps

- `apps/interface/` - Next.js web interface
- `apps/ink/` - Terminal UI and CLI
- `apps/py-server/` - Python FastAPI sidecar

### Key Integration Points

- [List external services, APIs, databases]

## Status

Current phase: [Setup / Development / Testing / Production]

## Quick Commands

```bash
pnpm setup     # First-time setup
pnpm dev       # Run all services
pnpm interface dev  # Just the web app
pnpm ink dev ui     # Just the TUI
```
```

## 6.3 Create plans folder

```bash
mkdir -p _context/plans
```

Create `_context/plans/README.md`:

```markdown
# Plans

Implementation plans and feature specifications live here.

## Naming Convention

- `YYYY-MM-feature-name.md` for dated plans
- `feature-name/` folder for multi-part plans
  - `overview.md`
  - `step-1.md`
  - `step-2.md`

## Template

```markdown
# Plan: [Feature Name]

## Goal

What are we trying to achieve?

## Context

Why now? What prompted this?

## Approach

How will we do it?

## Steps

1. [ ] Step one
2. [ ] Step two
3. [ ] Step three

## Open Questions

- Question 1?
- Question 2?

## Decision Log

- [Date]: Decided X because Y
```
```

## 6.4 Create decisions folder

```bash
mkdir -p _context/decisions
```

Create `_context/decisions/README.md`:

```markdown
# Decisions

Architecture Decision Records (ADRs) live here.

## Format

Each decision is a markdown file: `NNN-title.md`

## Template

```markdown
# ADR-NNN: [Title]

## Status

[Proposed | Accepted | Deprecated | Superseded]

## Context

What is the issue that we're seeing that is motivating this decision?

## Decision

What is the change that we're proposing and/or doing?

## Consequences

What becomes easier or more difficult to do because of this change?
```

## Example Decisions

- `001-use-pnpm-workspaces.md`
- `002-python-not-in-pnpm.md`
- `003-context-folder-structure.md`
```

## 6.5 Create dev folder

```bash
mkdir -p _context/dev
```

Create `_context/dev/README.md`:

```markdown
# Dev Documentation

API docs, SDK references, and development guides.

## Structure

```
dev/
  anthropic/     # Claude API, Agent SDK docs
  openai/        # OpenAI API docs
  vercel/        # Next.js, AI SDK docs
  [vendor]/      # Other vendor docs
```

## Why Keep Docs Here?

- Faster than web fetching during development
- Can be annotated with project-specific notes
- Available offline
- Provides context for AI assistants

## Updating

When APIs change, update the relevant docs here.
```

## 6.6 Create readings folder (for learning projects)

```bash
mkdir -p _context/readings
```

Create `_context/readings/README.md`:

```markdown
# Readings

Notes on readings, research, and sources.

## Format

- `author-title.md` - Notes on a single work
- `topic/` - Folder for related readings

## Template

```markdown
# [Title] by [Author]

## Summary

[Brief summary]

## Key Ideas

- Idea 1
- Idea 2
- Idea 3

## Quotes

> "Notable quote" (p. XX)

## Questions

- Question this raised?

## Connections

- How does this connect to other readings?
- How does it connect to the project?
```
```

## 6.7 File naming conventions

| Type | Convention | Example |
|------|------------|---------|
| Setup docs | `NN-name.md` | `01-initial-setup.md` |
| Plans | `YYYY-MM-name.md` | `2024-12-voice-triggers.md` |
| Decisions | `NNN-name.md` | `001-use-pnpm.md` |
| Readings | `author-title.md` | `hegel-phenomenology.md` |
| Dev docs | `vendor/topic.md` | `anthropic/tool-use.md` |

## Next Steps

- [07-claude-md.md](07-claude-md.md) - Create the CLAUDE.md file for AI assistants
