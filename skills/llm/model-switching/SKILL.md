---
name: model-switching
description: Use when switching between Ollama models in an agent - loading models, configuring fallback, and dynamically changing LLM providers
---

# Model Switching

## Overview

Switch between different Ollama models (Llama 3, Mistral, etc.) dynamically or based on task requirements.

## When to Use

- Using different models for different tasks
- Adding fallback models
- Testing model performance
- Optimizing cost/speed tradeoffs

## Available Models

| Model | Strengths | Best For |
|-------|-----------|----------|
| llama3 | General purpose, good reasoning | Most tasks |
| mistral | Fast, efficient | Simple tasks |
| codellama | Code generation | Programming tasks |
| phi | Very fast, smaller | Quick responses |
| mixtral | Excellent reasoning | Complex tasks |

## Basic Model Switching

```python
from langchain_openai import ChatOpenAI

MODELS = {
    "fast": "phi",
    "balanced": "llama3",
    "powerful": "mixtral",
    "code": "codellama",
}

def get_llm(model_type: str = "balanced", **kwargs) -> ChatOpenAI:
    model_name = MODELS.get(model_type, "llama3")
    return ChatOpenAI(
        model=model_name,
        base_url="http://localhost:11434/v1",
        api_key="ollama",
        **kwargs
    )

llm = get_llm("fast", temperature=0.5)
```

## Dynamic Model Router

```python
from typing import Literal

class ModelRouter:
    def __init__(self):
        self.models = {
            "simple": {"model": "phi", "temperature": 0.3},
            "reasoning": {"model": "mixtral", "temperature": 0.7},
            "creative": {"model": "llama3", "temperature": 0.9},
            "code": {"model": "codellama", "temperature": 0.2},
        }
        self.default = "balanced"
    
    def get_llm(self, task_type: str = None) -> ChatOpenAI:
        config = self.models.get(task_type, self.models[self.default])
        return ChatOpenAI(
            base_url="http://localhost:11434/v1",
            api_key="ollama",
            **config
        )
    
    def add_model(self, name: str, model: str, **kwargs):
        self.models[name] = {"model": model, **kwargs}

router = ModelRouter()
```

## Fallback Chain

```python
class FallbackChain:
    def __init__(self):
        self.chain = []
    
    def add_fallback(self, model: str, **kwargs):
        self.chain.append({"model": model, **kwargs})
        return self
    
    async def invoke(self, prompt: str) -> str:
        errors = []
        
        for attempt, config in enumerate(self.chain):
            try:
                llm = ChatOpenAI(
                    base_url="http://localhost:11434/v1",
                    api_key="ollama",
                    **config
                )
                return await llm.ainvoke(prompt)
            except Exception as e:
                errors.append(f"{config['model']}: {str(e)}")
                if attempt < len(self.chain) - 1:
                    continue
        
        raise Exception(f"All models failed: {errors}")

fallback = (
    FallbackChain()
    .add_fallback("mixtral")
    .add_fallback("llama3")
    .add_fallback("phi")
)
```

## Task-Based Selection

```python
def select_model_for_task(task: str) -> str:
    """Select appropriate model based on task type."""
    task_lower = task.lower()
    
    if any(word in task_lower for word in ["code", "function", "debug", "script"]):
        return "codellama"
    
    if any(word in task_lower for word in ["explain", "analyze", "compare", "why"]):
        return "mixtral"
    
    if any(word in task_lower for word in ["quick", "simple", "list"]):
        return "phi"
    
    return "llama3"

task_model = select_model_for_task("Write a Python function")
llm = get_llm(task_model)
```

## Model Configuration Presets

```python
PRESETS = {
    "conservative": {
        "model": "llama3",
        "temperature": 0.1,
        "max_tokens": 500,
        "top_p": 0.9,
    },
    "creative": {
        "model": "llama3",
        "temperature": 0.9,
        "max_tokens": 1000,
        "top_p": 0.95,
    },
    "precise": {
        "model": "mixtral",
        "temperature": 0.0,
        "max_tokens": 800,
        "top_p": 0.9,
    },
    "quick": {
        "model": "phi",
        "temperature": 0.5,
        "max_tokens": 200,
    },
}

def apply_preset(preset_name: str) -> ChatOpenAI:
    config = PRESETS.get(preset_name, PRESETS["conservative"])
    return ChatOpenAI(
        base_url="http://localhost:11434/v1",
        api_key="ollama",
        **config
    )
```

## Ollama Model Management

```bash
# List installed models
ollama list

# Pull new model
ollama pull mistral

# Remove model
ollama rm phi

# Copy model (create variant)
ollama cp llama3 llama3-instruction

# Show model info
ollama show llama3
```

## Model Switching in LangGraph

```python
from langgraph.graph import StateGraph

class AgentState(TypedDict):
    messages: list
    current_model: str

def routing_node(state: AgentState) -> AgentState:
    """Select model based on conversation."""
    query = state["messages"][-1].content
    model = select_model_for_task(query)
    return {"current_model": model}

def agent_node(state: AgentState) -> AgentState:
    """Run agent with selected model."""
    model = state["current_model"]
    llm = get_llm(model)
    response = llm.invoke(state["messages"])
    return {"messages": state["messages"] + [response]}

workflow = StateGraph(AgentState)
workflow.add_node("router", routing_node)
workflow.add_node("agent", agent_node)
workflow.add_edge("router", "agent")
```

## Monitoring Model Usage

```python
from datetime import datetime

class ModelUsageTracker:
    def __init__(self):
        self.usage = defaultdict(lambda: {"count": 0, "tokens": 0})
    
    def track(self, model: str, tokens: int):
        self.usage[model]["count"] += 1
        self.usage[model]["tokens"] += tokens
    
    def report(self) -> dict:
        return dict(self.usage)

tracker = ModelUsageTracker()
```

## Quick Reference

| Function | Purpose |
|----------|---------|
| `get_llm()` | Get model by type |
| `ModelRouter` | Route by task type |
| `FallbackChain` | Chain with fallbacks |
| `select_model_for_task` | Auto-select model |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Wrong model name | Verify with `ollama list` |
| No fallback | Always add fallback chain |
| Incompatible params | Check model capabilities |
| Memory issues | Pull models one at a time |

## Dependencies

```bash
pip install langchain langchain-openai
```

## Next Steps

- `ollama-integration` - Ollama setup
- `langgraph-agent-setup` - Use in agent
- `prompt-template-system` - Tune prompts per model
