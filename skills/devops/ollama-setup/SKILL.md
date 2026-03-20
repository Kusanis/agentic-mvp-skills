---
name: ollama-setup
description: Use when installing and configuring Ollama for local LLM inference - downloading models, verifying setup, and optimizing performance
---

# Ollama Setup

## Overview

Install, configure, and optimize Ollama for local LLM inference in agentic AI systems.

## When to Use

- First-time Ollama installation
- Downloading new models
- Troubleshooting Ollama issues
- Optimizing model performance

## Installation

### macOS/Linux

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh
```

### Windows

1. Download from https://ollama.com/download
2. Run the installer
3. Ollama will start automatically

### Verify Installation

```bash
ollama --version
```

## Start Ollama

```bash
# Start Ollama server (usually starts automatically)
ollama serve

# Check if running
curl http://localhost:11434/api/tags
```

## Download Models

```bash
# Download popular models
ollama pull llama3      # General purpose (~5GB)
ollama pull mistral     # Fast, efficient (~4GB)
ollama pull phi         # Very fast (~2GB)
ollama pull mixtral     # Excellent reasoning (~26GB)
ollama pull codellama   # Code generation (~4GB)

# List installed models
ollama list
```

## Model Requirements

| Model | Size | RAM | Best For |
|-------|------|-----|----------|
| phi | 2GB | 4GB+ | Quick tasks |
| mistral | 4GB | 8GB+ | Balanced |
| llama3 | 5GB | 8GB+ | General use |
| llama3:70b | 40GB | 64GB+ | Complex tasks |
| mixtral | 26GB | 32GB+ | Reasoning |

## Common Ollama Commands

```bash
# List models
ollama list

# Show model info
ollama show llama3

# Copy/create variant
ollama cp llama3 llama3-instruction

# Remove model
ollama rm phi

# Create custom model (Modelfile)
ollama create custom-model -f Modelfile
```

## Create Custom Model (Modelfile)

```dockerfile
# Modelfile
FROM llama3

PARAMETER temperature 0.7
PARAMETER top_p 0.9

SYSTEM """
You are a retail assistant. Help customers with:
- Product recommendations
- Order tracking
- Returns and refunds
"""

TEMPLATE """
[INST] <<SYS>>
{{ .System }}
<</SYS>>
{{ .Prompt }} [/INST]
"""
```

```bash
# Create custom model
ollama create retail-assistant -f Modelfile

# Use it
ollama run retail-assistant
```

## Test Ollama API

```bash
# Simple chat
curl http://localhost:11434/api/chat -d '{
  "model": "llama3",
  "messages": [{"role": "user", "content": "Hello"}],
  "stream": false
}'

# Generate embeddings
curl http://localhost:11434/api/embeddings -d '{
  "model": "llama3",
  "prompt": "Hello world"
}'

# List running models
curl http://localhost:11434/api/ps
```

## Ollama Configuration

```bash
# Environment variables
export OLLAMA_HOST=0.0.0.0        # Listen on all interfaces
export OLLAMA_PORT=11434          # Default port
export OLLAMA_MODELS=/path/to/models  # Custom model directory

# GPU acceleration (automatic on NVIDIA)
nvidia-smi  # Check GPU availability
```

## Ollama Python Client

```python
import ollama

# Chat
response = ollama.chat(model='llama3', messages=[
    {'role': 'user', 'content': 'Hello!'}
])
print(response['message']['content'])

# Generate
response = ollama.generate(model='llama3', prompt='Write a poem')
print(response['response'])

# Embeddings
response = ollama.embeddings(model='llama3', prompt='Hello world')
print(f"Embedding: {len(response['embedding'])} dims")
```

## Troubleshooting

### Ollama not starting

```bash
# Check logs
journalctl -u ollama  # Linux
# Check Console app  # macOS

# Kill and restart
pkill ollama
ollama serve
```

### Model download failing

```bash
# Check disk space
df -h

# Clear cache
rm -rf ~/.ollama/models

# Retry download
ollama pull llama3
```

### Out of memory

```bash
# Use smaller model
ollama pull phi

# Check memory
free -h  # Linux
# Activity Monitor  # macOS
```

### Slow responses

```bash
# Check GPU usage
nvidia-smi

# Use GPU acceleration
export OLLAMA_VULKAN=1

# Reduce context size
curl http://localhost:11434/api/chat -d '{
  "model": "llama3",
  "options": {"num_ctx": 2048}
}'
```

## Performance Optimization

```python
# Optimized client settings
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="llama3",
    base_url="http://localhost:11434/v1",
    api_key="ollama",
    timeout=120,  # Longer timeout
    max_retries=3,  # Retry on failure
)
```

## Quick Reference

| Task | Command |
|------|---------|
| Start | `ollama serve` |
| Download | `ollama pull <model>` |
| List | `ollama list` |
| Run | `ollama run <model>` |
| Remove | `ollama rm <model>` |
| Info | `ollama show <model>` |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Not enough RAM | Use smaller model (phi) |
| Download timeout | Check internet, retry |
| Wrong port | Default is 11434 |
| No GPU | Use CPU models or phi |

## Dependencies

```bash
# Python client
pip install ollama
```

## Next Steps

- `environment-setup` - Configure environment
- `ollama-integration` - Connect to agent
- `deployment-checklist` - Verify setup
