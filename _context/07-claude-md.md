# Step 7: CLAUDE.md Setup

Create the CLAUDE.md file that provides guidance to Claude Code and other AI assistants.

## 7.1 Create CLAUDE.md

Create `CLAUDE.md` in the repo root:

```markdown
# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

A Learning Machine is a monorepo template for learning, writing, and building projects. It combines a Next.js web interface, an Ink-based terminal UI, and a Python FastAPI sidecar.

## Commands

### Setup

```bash
pnpm setup          # First-time setup (installs deps, creates venvs)
```

### Development

```bash
pnpm dev            # Start all services (Next.js + Python)
pnpm interface dev  # Just the web interface
pnpm ink dev ui     # Launch the terminal UI
pnpm ink dev hello  # Run a CLI command
```

### Build

```bash
pnpm build          # Build all packages
pnpm typecheck      # Type check all packages
pnpm lint           # Lint all packages
```

### Python Server

```bash
cd apps/py-server
source .venv/bin/activate
uvicorn src.main:app --reload --port 8001
```

## Architecture

### Monorepo Structure (pnpm workspaces)

- `apps/interface/` - Next.js 15 web app with Tailwind and shadcn/ui
- `apps/ink/` - Ink/React terminal UI with Commander CLI
- `apps/py-server/` - Python FastAPI sidecar (NOT in pnpm workspace)
- `packages/` - Shared TypeScript packages (when needed)
- `scripts/` - Root-level shell scripts

### Context Folders

- `_context/` - LLM context docs, plans, decisions, readings
- `_output/` - Generated artifacts (gitignored)
- `_reference/` - Reference materials (gitignored)

### Ports

- `3000` - Next.js interface
- `8001` - Python FastAPI server

## Code Style

### General

- TypeScript for all Node.js code
- Python 3.11+ with type hints
- Use ESM (`"type": "module"`) in all packages

### Formatting

- Prettier for TypeScript/JavaScript
- Ruff for Python
- 2-space indentation for TS/JS, 4-space for Python

### ASCII Only in Documentation

Use only simple ASCII characters in markdown files and code comments. Avoid unicode box-drawing, arrows, or fancy quotes.

## Environment Variables

Check `.env.example` for required variables. Each app may have its own `.env` file:

- `apps/interface/.env.local` (Next.js convention)
- `apps/py-server/.env`

## Testing

```bash
# TypeScript tests (when added)
pnpm test

# Python tests
cd apps/py-server
source .venv/bin/activate
pytest
```

## Common Tasks

### Add a new API route (Python)

1. Create route file in `apps/py-server/src/routes/`
2. Register router in `apps/py-server/src/main.py`

### Add a new CLI command (Ink)

1. Add command in `apps/ink/src/cli.tsx`
2. Create component in `apps/ink/src/components/` if needed

### Add a shadcn component

```bash
cd apps/interface
pnpm dlx shadcn@latest add [component-name]
```

## File Naming

- Components: PascalCase (`Header.tsx`)
- Utilities: camelCase (`formatDate.ts`)
- Routes/pages: kebab-case folders (`/api/hello-world`)
- Python: snake_case (`my_service.py`)
```

## 7.2 Alternative: AGENTS.md

Some teams prefer `AGENTS.md` as a more general name. The content is the same - use whichever you prefer. Claude Code looks for `CLAUDE.md` specifically, but will also use context from any markdown files.

## 7.3 Keep it updated

The CLAUDE.md file should evolve with your project:

- Add new commands as you create them
- Document architectural decisions
- Note any gotchas or non-obvious patterns
- List environment variables as they're added

## What NOT to put in CLAUDE.md

- Sensitive information (API keys, secrets)
- Verbose documentation (link to `_context/` instead)
- Implementation details that change frequently

## Next Steps

You now have a complete learning machine setup!

Optional next steps:
- [08-docs-setup.md](08-docs-setup.md) - Add a docs/content route to Next.js
- [09-slack-bot-setup.md](09-slack-bot-setup.md) - Add a Slack bot app
- [10-shared-packages.md](10-shared-packages.md) - Create shared TypeScript packages
