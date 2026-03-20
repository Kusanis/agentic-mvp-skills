---
name: langgraph-agent-setup
description: Use when initializing a new LangGraph agent - setting up state schema, defining agent identity, and preparing the graph structure for AI agent systems
---

# LangGraph Agent Setup

## Overview

Initialize a LangGraph agent with proper state schema, tools, and graph structure. This is the foundation for building AI agent systems.

## When to Use

- Creating a new agent from scratch
- Defining what data the agent tracks during execution
- Setting up the agent's identity and capabilities

## Setup Checklist

1. [ ] Define state schema (TypedDict or Pydantic)
2. [ ] Create initial state values
3. [ ] Define agent node function
4. [ ] Create tool binding (see `langgraph-tool-binding`)
5. [ ] Build the graph
6. [ ] Compile the agent

## State Schema Definition

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolNode

class AgentState(TypedDict):
    """State schema for the agent."""
    messages: list
    current_step: str
    context: dict
    tools_used: list[str]
```

## Agent Node Function

```python
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage

def agent_node(state: AgentState, config: dict) -> AgentState:
    """Main agent node that processes state and returns decisions."""
    llm = ChatOpenAI(
        model="llama3",
        base_url="http://localhost:11434/v1",
        api_key="ollama"
    )
    
    system_prompt = """You are a helpful AI agent.
    Use tools when needed to complete tasks."""
    
    response = llm.invoke([
        {"role": "system", "content": system_prompt},
        *state["messages"]
    ])
    
    return {
        "messages": state["messages"] + [response],
        "current_step": "reasoning",
        "context": state.get("context", {}),
        "tools_used": state.get("tools_used", [])
    }
```

## Build the Graph

```python
from langgraph.graph import StateGraph

workflow = StateGraph(AgentState)

workflow.add_node("agent", agent_node)
workflow.add_node("tools", tool_node)

workflow.set_entry_point("agent")

workflow.add_conditional_edges(
    "agent",
    should_continue,
    {"continue": "agent", "end": END}
)

workflow.add_edge("tools", "agent")

agent = workflow.compile()
```

## Quick Reference

| Component | Purpose |
|-----------|---------|
| `AgentState` | TypedDict defining agent memory |
| `agent_node` | Main reasoning logic |
| `ToolNode` | Handles tool execution |
| `StateGraph` | Graph builder |
| `.compile()` | Creates executable agent |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting to add tools to state | Include `tools_used` in state schema |
| No message history | Keep `messages` list in state |
| Missing END node | Add conditional edges properly |

## Next Steps

After setup, use:
- `langgraph-tool-binding` - Bind Python functions as tools
- `langgraph-execution-flow` - Control agent decision loops
- `fastapi-agent-endpoints` - Expose agent via API
