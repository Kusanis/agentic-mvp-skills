---
name: langgraph-tool-binding
description: Use when binding Python functions as tools to a LangGraph agent - creating tool schemas, registering functions, and enabling agents to execute actions
---

# LangGraph Tool Binding

## Overview

Bind Python functions as tools that LangGraph agents can call. Tools extend agent capabilities beyond just reasoning.

## When to Use

- Adding custom functionality to an agent
- Connecting agent to external systems (APIs, databases)
- Enabling agent to perform actions (not just respond)

## Tool Binding Checklist

1. [ ] Define tool function with docstring
2. [ ] Create tool schema using `@tool` decorator
3. [ ] Bind tools to LLM
4. [ ] Create ToolNode
5. [ ] Add tool node to graph

## Basic Tool Definition

```python
from langchain_core.tools import tool
from pydantic import BaseModel, Field

@tool
def calculate_discount(price: float, discount_percent: float) -> dict:
    """Calculate discounted price for a product.
    
    Args:
        price: Original price in dollars
        discount_percent: Discount percentage (0-100)
    
    Returns:
        Dictionary with original, discount, and final price
    """
    discount_amount = price * (discount_percent / 100)
    final_price = price - discount_amount
    return {
        "original_price": price,
        "discount_amount": discount_amount,
        "final_price": final_price
    }
```

## Tool with Structured Output

```python
from typing import Optional

class InventoryQuery(BaseModel):
    product_id: str = Field(description="Product identifier")
    warehouse: Optional[str] = Field(default=None, description="Warehouse location")

@tool(args_schema=InventoryQuery)
def check_inventory(product_id: str, warehouse: Optional[str] = None) -> dict:
    """Query current inventory levels for a product."""
    # Implementation here
    return {"product_id": product_id, "quantity": 150, "warehouse": warehouse or "main"}
```

## Bind Tools to LLM

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="llama3",
    base_url="http://localhost:11434/v1",
    api_key="ollama"
)

llm_with_tools = llm.bind_tools([calculate_discount, check_inventory])
```

## Create ToolNode

```python
from langgraph.prebuilt import ToolNode

tools = [calculate_discount, check_inventory]
tool_node = ToolNode(tools)
```

## Add to Graph

```python
workflow.add_node("tools", tool_node)

workflow.add_edge("tools", "agent")
```

## Tool Response Handling

```python
def agent_node(state: AgentState) -> AgentState:
    """Process tool calls in last message."""
    last_message = state["messages"][-1]
    
    if hasattr(last_message, "tool_calls"):
        return {"current_step": "executing_tools"}
    
    return {"current_step": "responding"}
```

## Quick Reference

| Pattern | Use Case |
|---------|----------|
| `@tool` decorator | Simple functions |
| `@tool(args_schema=...)` | Functions with validation |
| `bind_tools()` | Attach tools to LLM |
| `ToolNode` | Execute tools from state |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Missing docstring | Agent can't use tool without description |
| No return type hints | Use Pydantic for structured returns |
| Tools not in ToolNode | Always wrap tools in ToolNode |

## Example: Retail Agent Tools

```python
@tool
def check_order_status(order_id: str) -> dict:
    """Check the status of a customer order."""
    return {"order_id": order_id, "status": "shipped", "eta": "2 days"}

@tool
def process_refund(order_id: str, reason: str) -> dict:
    """Process a refund for a returned order."""
    return {"order_id": order_id, "refund_id": "REF-123", "status": "processed"}
```

## Next Steps

- `langgraph-execution-flow` - Control tool execution flow
- `chromadb-integration` - Add memory tools
- `fastapi-agent-endpoints` - Expose tools via API
