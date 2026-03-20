---
name: deployment-checklist
description: Use when verifying a local agentic AI MVP deployment - checking all services, testing connectivity, and ensuring components work together
---

# Deployment Checklist

## Overview

Verify that all components of the agentic AI MVP are working correctly together.

## When to Use

- Before running the application
- After initial setup
- Troubleshooting deployment issues
- Pre-deployment verification

## Pre-Flight Checklist

```python
def deployment_checklist():
    """Run all deployment checks."""
    checks = [
        ("Python Version", check_python_version),
        ("Dependencies", verify_dependencies),
        ("Ollama Service", check_ollama),
        ("Ollama Models", check_ollama_models),
        ("ChromaDB", check_chromadb),
        ("Database", check_sqlite),
        ("API Endpoints", check_api),
        ("WebSocket", check_websocket),
    ]
    
    results = []
    for name, check in checks:
        try:
            result = check()
            results.append((name, "✅ PASS", result))
        except Exception as e:
            results.append((name, "❌ FAIL", str(e)))
    
    print("\n" + "="*50)
    print("DEPLOYMENT CHECKLIST")
    print("="*50)
    
    all_passed = True
    for name, status, detail in results:
        print(f"{name}: {status}")
        if "FAIL" in status:
            all_passed = False
            print(f"  → {detail}")
    
    print("="*50)
    print(f"Overall: {'✅ ALL CHECKS PASSED' if all_passed else '❌ SOME CHECKS FAILED'}")
    print("="*50)
    
    return all_passed
```

## 1. Python Version Check

```python
def check_python_version() -> str:
    import sys
    required = (3, 10)
    if sys.version_info < required:
        raise Exception(f"Python {required[0]}.{required[1]}+ required")
    return f"Python {sys.version_info.major}.{sys.version_info.minor}.{sys.version_info.micro}"
```

## 2. Dependencies Check

```python
def verify_dependencies() -> str:
    required_packages = [
        "fastapi",
        "langgraph",
        "langchain",
        "chromadb",
        "pydantic",
        "uvicorn",
        "httpx",
    ]
    
    missing = []
    for pkg in required_packages:
        try:
            __import__(pkg)
        except ImportError:
            missing.append(pkg)
    
    if missing:
        raise Exception(f"Missing: {', '.join(missing)}")
    
    return f"All {len(required_packages)} packages installed"
```

## 3. Ollama Service Check

```python
import requests

def check_ollama() -> str:
    try:
        response = requests.get("http://localhost:11434/api/tags", timeout=5)
        if response.status_code == 200:
            return "Ollama service running"
    except requests.exceptions.ConnectionError:
        raise Exception("Ollama not running. Start with: ollama serve")
    except Exception as e:
        raise Exception(f"Ollama error: {e}")
```

## 4. Ollama Models Check

```python
def check_ollama_models() -> str:
    response = requests.get("http://localhost:11434/api/tags")
    models = response.json().get("models", [])
    
    if not models:
        raise Exception("No models installed. Run: ollama pull llama3")
    
    model_names = [m["name"] for m in models]
    return f"Models available: {', '.join(model_names)}"
```

## 5. ChromaDB Check

```python
def check_chromadb() -> str:
    import chromadb
    from pathlib import Path
    
    chroma_dir = Path("./data/chroma_db")
    client = chromadb.PersistentClient(path=str(chroma_dir))
    
    collections = client.list_collections()
    return f"ChromaDB ready ({len(collections)} collections)"
```

## 6. SQLite Check

```python
import sqlite3
from pathlib import Path

def check_sqlite() -> str:
    db_path = Path("./data/memory.db")
    db_path.parent.mkdir(parents=True, exist_ok=True)
    
    conn = sqlite3.connect(db_path)
    cursor = conn.execute("SELECT 1")
    conn.close()
    
    return f"SQLite ready at {db_path}"
```

## 7. API Endpoints Check

```python
import httpx

def check_api() -> str:
    async def verify():
        async with httpx.AsyncClient() as client:
            response = await client.get("http://localhost:8000/health", timeout=5)
            if response.status_code == 200:
                return "FastAPI running"
            raise Exception(f"Status: {response.status_code}")
    
    try:
        import asyncio
        return asyncio.run(verify())
    except httpx.ConnectError:
        raise Exception("FastAPI not running. Start with: uvicorn app:app")
```

## 8. WebSocket Check

```python
def check_websocket() -> str:
    import websockets
    import asyncio
    
    async def verify():
        try:
            async with websockets.connect("ws://localhost:8000/ws/agent/test") as ws:
                return "WebSocket endpoint accessible"
        except Exception as e:
            raise Exception(f"WebSocket error: {e}")
    
    try:
        import asyncio
        return asyncio.run(verify())
    except:
        return "WebSocket endpoint not configured"
```

## Manual Verification Steps

```bash
# 1. Test Ollama API
curl http://localhost:11434/api/chat -d '{
  "model": "llama3",
  "messages": [{"role": "user", "content": "test"}],
  "stream": false
}'

# 2. Test FastAPI
curl http://localhost:8000/health

# 3. Test agent endpoint
curl -X POST http://localhost:8000/agent/invoke \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello"}'

# 4. Check frontend
curl http://localhost:3000
```

## Quick Reference

| Check | Command |
|-------|---------|
| Ollama | `curl localhost:11434/api/tags` |
| API | `curl localhost:8000/health` |
| Models | `ollama list` |
| Logs | `tail -f backend.log` |

## Common Issues

| Issue | Solution |
|-------|----------|
| Ollama not running | `ollama serve` |
| No models | `ollama pull llama3` |
| API timeout | Increase timeout value |
| Port conflict | Change port in .env |

## Next Steps

After all checks pass:
1. Start frontend: `cd frontend && npm run dev`
2. Open browser to configured URL
3. Test end-to-end conversation
4. Monitor logs for errors

## Deployment Report

```python
def generate_report():
    """Generate deployment status report."""
    report = {
        "timestamp": datetime.now().isoformat(),
        "python_version": sys.version,
        "ollama": {
            "service": is_ollama_running(),
            "models": get_installed_models(),
        },
        "databases": {
            "chromadb": is_chromadb_ready(),
            "sqlite": is_sqlite_ready(),
        },
        "api": {
            "fastapi": is_fastapi_running(),
            "websocket": is_websocket_ready(),
        }
    }
    
    with open("deployment_report.json", "w") as f:
        json.dump(report, f, indent=2)
    
    return report
```
