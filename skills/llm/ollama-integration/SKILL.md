---
name: ollama-integration
description: Use when connecting FastAPI to local Ollama API for LLM inference - setting up the client, handling responses, and configuring Ollama models for agent use
---

# Ollama Integration

## Overview

Connect FastAPI/LangGraph to local Ollama API for LLM inference without external cloud providers.

## When to Use

- Setting up local LLM inference
- Configuring Ollama as LangChain backend
- Replacing OpenAI API with local models
- Running agents entirely offline

## Ollama Setup

```bash
# Install Ollama (macOS/Linux)
curl -fsSL https://ollama.com/install.sh | sh

# Windows: Download from https://ollama.com/download

# Pull a model
ollama pull llama3
ollama pull mistral

# Verify running
ollama list
```

## Basic Ollama API

```bash
# Chat completions
curl http://localhost:11434/api/chat -d '{
  "model": "llama3",
  "messages": [{"role": "user", "content": "Hello!"}]
}'

# Generate embeddings
curl http://localhost:11434/api/embeddings -d '{
  "model": "llama3",
  "prompt": "Hello world"
}'
```

## LangChain Ollama Setup

```bash
pip install langchain langchain-openai
```

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="llama3",
    base_url="http://localhost:11434/v1",
    api_key="ollama",
    temperature=0.7,
    max_tokens=500,
)

response = llm.invoke("What is the capital of France?")
print(response.content)
```

## Streaming Responses

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="llama3",
    base_url="http://localhost:11434/v1",
    api_key="ollama",
)

for chunk in llm.stream("Write a haiku about AI"):
    print(chunk.content, end="", flush=True)
```

## Ollama Client for Embeddings

```python
from langchain_community.embeddings import OllamaEmbeddings

embeddings = OllamaEmbeddings(
    model="llama3",
    base_url="http://localhost:11434/v1",
)

vector = embeddings.embed_query("Hello world")
print(f"Embedding dimension: {len(vector)}")
```

## Custom Ollama Client

```python
import requests
from typing import Optional

class OllamaClient:
    def __init__(self, base_url: str = "http://localhost:11434"):
        self.base_url = base_url
    
    def chat(self, model: str, messages: list, **kwargs) -> dict:
        response = requests.post(
            f"{self.base_url}/api/chat",
            json={
                "model": model,
                "messages": messages,
                **kwargs
            }
        )
        return response.json()
    
    def generate(self, model: str, prompt: str, **kwargs) -> dict:
        response = requests.post(
            f"{self.base_url}/api/generate",
            json={
                "model": model,
                "prompt": prompt,
                **kwargs
            }
        )
        return response.json()
    
    def embeddings(self, model: str, prompt: str) -> list:
        response = requests.post(
            f"{self.base_url}/api/embeddings",
            json={"model": model, "prompt": prompt}
        )
        return response.json()["embedding"]

ollama = OllamaClient()

response = ollama.chat("llama3", [
    {"role": "user", "content": "Hello!"}
])
print(response["message"]["content"])
```

## Async Ollama Client

```python
import httpx
import asyncio

class AsyncOllamaClient:
    def __init__(self, base_url: str = "http://localhost:11434"):
        self.base_url = base_url
    
    async def chat(self, model: str, messages: list, **kwargs) -> dict:
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{self.base_url}/api/chat",
                json={"model": model, "messages": messages, **kwargs}
            )
            return response.json()
    
    async def chat_stream(self, model: str, messages: list):
        async with httpx.AsyncClient() as client:
            async with client.stream(
                "POST",
                f"{self.base_url}/api/chat",
                json={"model": model, "messages": messages, "stream": True}
            ) as response:
                async for line in response.aiter_lines():
                    if line:
                        yield json.loads(line)

ollama = AsyncOllamaClient()

async def main():
    response = await ollama.chat("llama3", [
        {"role": "user", "content": "Hello!"}
    ])
    print(response["message"]["content"])

asyncio.run(main())
```

## Health Check

```python
def check_ollama_health() -> bool:
    """Check if Ollama is running and responsive."""
    try:
        response = requests.get("http://localhost:11434/api/tags")
        return response.status_code == 200
    except requests.exceptions.ConnectionError:
        return False

def get_available_models() -> list[str]:
    """Get list of installed Ollama models."""
    try:
        response = requests.get("http://localhost:11434/api/tags")
        if response.status_code == 200:
            data = response.json()
            return [m["name"] for m in data.get("models", [])]
    except:
        pass
    return []
```

## Quick Reference

| Task | Command |
|------|---------|
| Install model | `ollama pull llama3` |
| List models | `ollama list` |
| Chat API | `POST /api/chat` |
| Embeddings | `POST /api/embeddings` |
| Health check | `GET /api/tags` |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Model not pulled | Run `ollama pull <model>` first |
| Wrong API key | Use any string, Ollama ignores it |
| Wrong base URL | Use `http://localhost:11434/v1` |
| No streaming | Use `.stream()` method |

## Dependencies

```bash
pip install langchain langchain-openai requests httpx
```

## Next Steps

- `prompt-template-system` - Create structured prompts
- `model-switching` - Switch between models
- `langgraph-agent-setup` - Use Ollama in agent
