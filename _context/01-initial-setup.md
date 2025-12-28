# Step 1: Initial Setup

Set up an existing repo with the basic structure and gitignore for a learning-machine monorepo.

## Prerequisites

Make sure you have these installed:

```bash
pnpm -v
node -v
git -v
python3 -V
```

## 1. Create folder structure

From the repo root:

```bash
mkdir -p apps/{interface,ink,cli,py-server} packages scripts _output _reference
```

## 2. Create .gitignore

Create `.gitignore` in the repo root:

```gitignore
# OS
.DS_Store
Thumbs.db

# Dependencies
node_modules/
.pnpm-store/

# Build outputs
dist/
.next/
out/

# Environment files
.env
.env.local
.env.*.local
*.env
!.env.example

# Project folders (gitignored by convention)
_output/
_reference/

# TypeScript
*.tsbuildinfo

# Logs
*.log
logs/

# Python
__pycache__/
*.py[cod]
*.pyo
*.pyd
.pytest_cache/
.mypy_cache/
.venv/
venv/
ENV/
pip-wheel-metadata/
*.egg-info/

# Editor
.vscode/
.idea/
*.swp
*.swo

# Coverage
coverage/
```

## 3. Create pnpm-workspace.yaml

```yaml
packages:
  - "apps/*"
  - "!apps/py-server"
  - "packages/*"
```

Note: `py-server` is excluded because Python projects are managed separately (venv/uv/poetry), not by pnpm.

## 4. Create root package.json

```json
{
  "name": "a-learning-machine",
  "version": "0.1.0",
  "private": true,
  "packageManager": "pnpm@10.0.0",
  "engines": {
    "node": ">=18"
  },
  "scripts": {
    "dev": "pnpm -r --parallel dev",
    "build": "pnpm -r build",
    "lint": "pnpm -r lint",
    "typecheck": "pnpm -r typecheck"
  }
}
```

## 5. Add placeholder READMEs

Create `_output/README.md`:

```markdown
# _output

Generated artifacts go here (exports, PDFs, data dumps, rendered media).

This folder is gitignored by default.
```

Create `_reference/README.md`:

```markdown
# _reference

Source materials go here (papers, PDFs, transcripts, vendor docs).

This folder is gitignored by default.
```

## 6. Verify

```bash
git status
```

You should see:
- `.gitignore`
- `pnpm-workspace.yaml`
- `package.json`
- New folders created

## Next Steps

- [02-next-interface-setup.md](02-next-interface-setup.md) - Set up the Next.js interface app
- [03-ink-cli-setup.md](03-ink-cli-setup.md) - Set up the Ink TUI/CLI
- [04-py-server-setup.md](04-py-server-setup.md) - Set up the FastAPI Python server
