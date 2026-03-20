---
name: environment-setup
description: Use when setting up the local development environment for an agentic AI MVP - Python environment, dependencies, and configuration management
---

# Environment Setup

## Overview

Set up the Python environment and manage dependencies for building agentic AI systems.

## When to Use

- Initial project setup
- Installing dependencies
- Configuring environment variables
- Setting up virtual environments

## Project Structure

```
agentic-mvp/
├── backend/
│   ├── app.py              # FastAPI application
│   ├── agent/              # LangGraph agent code
│   ├── tools/              # Custom tools
│   ├── memory/             # Memory modules
│   └── requirements.txt
├── frontend/
│   ├── src/
│   ├── package.json
│   └── ...
├── data/
│   ├── chroma_db/          # Vector database
│   └── memory.db           # SQLite memory
└── .env
```

## Virtual Environment Setup

```bash
# Create virtual environment
python -m venv venv

# Activate (Windows)
venv\Scripts\activate

# Activate (Linux/macOS)
source venv/bin/activate
```

## Backend Dependencies

```bash
# requirements.txt
fastapi==0.109.0
uvicorn[standard]==0.27.0
langgraph==0.0.20
langchain==0.1.0
langchain-openai==0.0.5
langchain-community==0.0.10
langchain-chroma==0.1.0
chromadb==0.4.22
pydantic==2.5.0
python-dotenv==1.0.0
httpx==0.26.0
sse-starlette==1.8.0
```

## Install Dependencies

```bash
pip install -r requirements.txt
```

## Environment Variables (.env)

```bash
# .env.example

# Ollama
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=llama3

# ChromaDB
CHROMA_PERSIST_DIR=./data/chroma_db

# SQLite
SQLITE_DB_PATH=./data/memory.db

# API
API_HOST=0.0.0.0
API_PORT=8000

# Frontend
FRONTEND_URL=http://localhost:3000
```

## Load Environment Variables

```python
from dotenv import load_dotenv
from pathlib import Path

env_path = Path(__file__).parent.parent / ".env"
load_dotenv(env_path)

import os

OLLAMA_URL = os.getenv("OLLAMA_BASE_URL", "http://localhost:11434")
MODEL = os.getenv("OLLAMA_MODEL", "llama3")
```

## Directory Creation Script

```python
from pathlib import Path

def setup_directories():
    """Create necessary directories for the project."""
    dirs = [
        "data",
        "data/chroma_db",
        "backend/agent",
        "backend/tools",
        "backend/memory",
        "frontend/src/components",
    ]
    
    for d in dirs:
        Path(d).mkdir(parents=True, exist_ok=True)
    
    print("Directories created successfully")

if __name__ == "__main__":
    setup_directories()
```

## Python Version Check

```python
import sys

MIN_PYTHON_VERSION = (3, 10)

def check_python_version():
    if sys.version_info < MIN_PYTHON_VERSION:
        print(f"Python {MIN_PYTHON_VERSION[0]}.{MIN_PYTHON_VERSION[1]}+ required")
        sys.exit(1)
    print(f"Python {sys.version_info.major}.{sys.version_info.minor} - OK")

check_python_version()
```

## Dependency Verification

```python
def verify_dependencies():
    """Verify all required packages are installed."""
    required = [
        "fastapi",
        "langgraph",
        "chromadb",
        "pydantic",
    ]
    
    missing = []
    for package in required:
        try:
            __import__(package)
        except ImportError:
            missing.append(package)
    
    if missing:
        print(f"Missing packages: {', '.join(missing)}")
        print("Run: pip install -r requirements.txt")
        return False
    
    print("All dependencies installed - OK")
    return True

verify_dependencies()
```

## Startup Script

```bash
# start.sh
#!/bin/bash

# Start Ollama (if not running)
if ! curl -s http://localhost:11434 > /dev/null; then
    echo "Starting Ollama..."
    ollama serve &
    sleep 3
fi

# Set environment
export OLLAMA_MODEL=${OLLAMA_MODEL:-llama3}

# Start backend
cd backend
uvicorn app:app --reload --port 8000 &

# Start frontend
cd ../frontend
npm run dev
```

## Quick Reference

| Task | Command |
|------|---------|
| Create venv | `python -m venv venv` |
| Install deps | `pip install -r requirements.txt` |
| Load env | `python-dotenv` |
| Check Python | `python --version` |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Wrong Python version | Use 3.10+ |
| Missing .env | Copy from .env.example |
| Ollama not running | Start with `ollama serve` |
| Permissions | Use virtual environment |

## Next Steps

- `ollama-setup` - Install and configure Ollama
- `deployment-checklist` - Verify everything works
- `langgraph-agent-setup` - Build first agent
