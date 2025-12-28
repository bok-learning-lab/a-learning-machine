# Step 4: Python FastAPI Server Setup

Set up a FastAPI sidecar service for Python-specific tasks.

Note: This is NOT managed by pnpm. It uses its own virtual environment.

## 4.1 Create the folder structure

```bash
mkdir -p apps/py-server/src/routes
mkdir -p apps/py-server/src/services
mkdir -p apps/py-server/scripts
```

## 4.2 Create pyproject.toml

Create `apps/py-server/pyproject.toml`:

```toml
[project]
name = "a-learning-machine-py-server"
version = "0.1.0"
description = "Python sidecar service for A Learning Machine"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.115.0",
    "uvicorn[standard]>=0.32.0",
    "python-dotenv>=1.0.0",
    "pydantic>=2.6.0",
]

[project.optional-dependencies]
dev = [
    "ruff>=0.8.0",
    "pytest>=8.0.0",
    "httpx>=0.27.0",
]

[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "UP"]
```

## 4.3 Create .python-version

Create `apps/py-server/.python-version`:

```
3.11
```

## 4.4 Create the main FastAPI app

Create `apps/py-server/src/main.py`:

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from dotenv import load_dotenv
import os

load_dotenv()

app = FastAPI(
    title="A Learning Machine - Python Server",
    version="0.1.0",
)

# Allow CORS for local development
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.get("/")
def root():
    return {"message": "A Learning Machine Python Server"}


@app.get("/health")
def health():
    return {"status": "ok"}
```

## 4.5 Create a sample route

Create `apps/py-server/src/routes/echo.py`:

```python
from fastapi import APIRouter
from pydantic import BaseModel

router = APIRouter(prefix="/echo", tags=["echo"])


class EchoRequest(BaseModel):
    message: str


class EchoResponse(BaseModel):
    original: str
    reversed: str
    length: int


@router.post("/", response_model=EchoResponse)
def echo(request: EchoRequest):
    return EchoResponse(
        original=request.message,
        reversed=request.message[::-1],
        length=len(request.message),
    )
```

Then register it in `apps/py-server/src/main.py`:

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from dotenv import load_dotenv
from src.routes import echo  # Add this import

load_dotenv()

app = FastAPI(
    title="A Learning Machine - Python Server",
    version="0.1.0",
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(echo.router)  # Add this line


@app.get("/")
def root():
    return {"message": "A Learning Machine Python Server"}


@app.get("/health")
def health():
    return {"status": "ok"}
```

## 4.6 Create __init__.py files

Create `apps/py-server/src/__init__.py`:

```python
# empty
```

Create `apps/py-server/src/routes/__init__.py`:

```python
# empty
```

Create `apps/py-server/src/services/__init__.py`:

```python
# empty
```

## 4.7 Create dev script

Create `apps/py-server/scripts/dev.sh`:

```bash
#!/bin/bash
set -e

cd "$(dirname "$0")/.."

if [ ! -d ".venv" ]; then
    echo "Creating virtual environment..."
    python3 -m venv .venv
fi

source .venv/bin/activate

echo "Installing dependencies..."
pip install -e ".[dev]" --quiet

echo "Starting server..."
uvicorn src.main:app --reload --port 8001
```

Make it executable:

```bash
chmod +x apps/py-server/scripts/dev.sh
```

## 4.8 Create .env.example

Create `apps/py-server/.env.example`:

```bash
# Python server configuration
PORT=8001

# Add your API keys and secrets here
# OPENAI_API_KEY=
# ANTHROPIC_API_KEY=
```

## 4.9 Create README

Create `apps/py-server/README.md`:

```markdown
# py-server

Python FastAPI sidecar service.

## Setup

```bash
# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -e ".[dev]"
```

## Run

```bash
# Option 1: Use the dev script
./scripts/dev.sh

# Option 2: Run manually
source .venv/bin/activate
uvicorn src.main:app --reload --port 8001
```

## API Docs

Once running, visit:
- http://localhost:8001/docs (Swagger UI)
- http://localhost:8001/redoc (ReDoc)
```

## Setup and run

```bash
cd apps/py-server

# Create venv and install
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"

# Run
uvicorn src.main:app --reload --port 8001
```

Or just:

```bash
./apps/py-server/scripts/dev.sh
```

Open http://localhost:8001/docs for the Swagger UI.

## Next Steps

- [05-scripts-and-dev.md](05-scripts-and-dev.md) - Set up root scripts and dev orchestration
