# Step 5: Root Scripts and Dev Orchestration

Set up scripts to run all services together.

## 5.1 Create scripts folder

```bash
mkdir -p scripts
```

## 5.2 Create setup script

Create `scripts/setup.sh`:

```bash
#!/bin/bash
set -e

echo "=== A Learning Machine Setup ==="
echo ""

# Check prerequisites
echo "Checking prerequisites..."
command -v pnpm >/dev/null 2>&1 || { echo "pnpm is required but not installed."; exit 1; }
command -v node >/dev/null 2>&1 || { echo "node is required but not installed."; exit 1; }
command -v python3 >/dev/null 2>&1 || { echo "python3 is required but not installed."; exit 1; }

echo "Prerequisites OK"
echo ""

# Install Node dependencies
echo "Installing Node dependencies..."
pnpm install

# Set up Python environment
echo ""
echo "Setting up Python environment..."
cd apps/py-server

if [ ! -d ".venv" ]; then
    python3 -m venv .venv
fi

source .venv/bin/activate
pip install -e ".[dev]" --quiet
deactivate

cd ../..

# Copy env files if they don't exist
echo ""
echo "Checking environment files..."

if [ -f ".env.example" ] && [ ! -f ".env" ]; then
    cp .env.example .env
    echo "Created .env from .env.example"
fi

for env_example in $(find apps -name ".env.example" 2>/dev/null); do
    target="${env_example%.example}"
    if [ ! -f "$target" ]; then
        cp "$env_example" "$target"
        echo "Created $target"
    fi
done

echo ""
echo "=== Setup complete! ==="
echo ""
echo "Next steps:"
echo "  pnpm dev         - Run all services"
echo "  pnpm interface   - Run just the web interface"
echo "  pnpm ink         - Run the CLI"
```

Make executable:

```bash
chmod +x scripts/setup.sh
```

## 5.3 Create dev script

Create `scripts/dev.sh`:

```bash
#!/bin/bash

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Store PIDs for cleanup
PIDS=()

cleanup() {
    echo ""
    echo -e "${YELLOW}Shutting down services...${NC}"
    for pid in "${PIDS[@]}"; do
        kill "$pid" 2>/dev/null
    done
    exit 0
}

trap cleanup INT TERM

echo -e "${GREEN}=== Starting A Learning Machine ===${NC}"
echo ""

# Start Python server
if [ -d "apps/py-server" ]; then
    echo -e "${BLUE}Starting Python server on :8001...${NC}"
    (
        cd apps/py-server
        source .venv/bin/activate 2>/dev/null || {
            python3 -m venv .venv
            source .venv/bin/activate
            pip install -e ".[dev]" --quiet
        }
        uvicorn src.main:app --reload --port 8001 2>&1 | sed 's/^/[py-server] /'
    ) &
    PIDS+=($!)
fi

# Start Next.js interface
if [ -d "apps/interface" ]; then
    echo -e "${BLUE}Starting Next.js on :3000...${NC}"
    (
        cd apps/interface
        pnpm dev 2>&1 | sed 's/^/[interface] /'
    ) &
    PIDS+=($!)
fi

echo ""
echo -e "${GREEN}Services starting...${NC}"
echo -e "  Interface: ${BLUE}http://localhost:3000${NC}"
echo -e "  Python API: ${BLUE}http://localhost:8001${NC}"
echo -e "  API Docs: ${BLUE}http://localhost:8001/docs${NC}"
echo ""
echo -e "${YELLOW}Press Ctrl+C to stop all services${NC}"
echo ""

wait
```

Make executable:

```bash
chmod +x scripts/dev.sh
```

## 5.4 Update root package.json

Update the scripts in your root `package.json`:

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
    "setup": "./scripts/setup.sh",
    "dev": "./scripts/dev.sh",
    "build": "pnpm -r build",
    "lint": "pnpm -r lint",
    "typecheck": "pnpm -r typecheck",
    "interface": "pnpm --filter @a-learning-machine/interface",
    "ink": "pnpm --filter @a-learning-machine/ink"
  }
}
```

## 5.5 Create root .env.example

Create `.env.example` in repo root:

```bash
# A Learning Machine - Environment Variables
# Copy this to .env and fill in your values

# API Keys
# OPENAI_API_KEY=
# ANTHROPIC_API_KEY=

# Paths (defaults work for most cases)
OUTPUT_FOLDER=./_output
REFERENCE_FOLDER=./_reference
```

## Usage

### First time setup

```bash
pnpm setup
```

### Start all services

```bash
pnpm dev
```

### Run individual apps

```bash
# Next.js interface
pnpm interface dev

# Ink CLI
pnpm ink dev ui

# Python server only
./apps/py-server/scripts/dev.sh
```

## Next Steps

- [06-context-structure.md](06-context-structure.md) - Set up the _context folder conventions
