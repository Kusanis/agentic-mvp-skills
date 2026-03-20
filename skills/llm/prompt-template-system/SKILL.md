---
name: prompt-template-system
description: Use when creating structured prompts for LangGraph agents - system prompts, tool prompts, and context-aware templates for consistent agent behavior
---

# Prompt Template System

## Overview

Create structured, reusable prompts for LangGraph agents that ensure consistent behavior across different use cases.

## When to Use

- Defining agent personality/role
- Creating tool instructions
- Building context-aware responses
- Standardizing agent behavior

## Prompt Template Types

| Type | Purpose |
|------|---------|
| System | Agent identity and behavior |
| Tool | How to use specific tools |
| User | Task instructions |
| Few-shot | Example demonstrations |

## System Prompt Template

```python
from langchain.prompts import ChatPromptTemplate, SystemMessagePromptTemplate, HumanMessagePromptTemplate

SYSTEM_PROMPT = """You are a {agent_role}, helping users with {domain}.
Your expertise includes: {expertise}
Communication style: {style}
Always prioritize: {priorities}"""

system_template = SystemMessagePromptTemplate.from_template(SYSTEM_PROMPT)

prompt = ChatPromptTemplate.from_messages([
    system_template,
    HumanMessagePromptTemplate.from_template("{user_input}")
])

# Usage
formatted_prompt = prompt.format(
    agent_role="retail assistant",
    domain="online shopping",
    expertise="product recommendations, order tracking, returns",
    style="friendly and helpful",
    priorities="accuracy, customer satisfaction",
    user_input="Where is my order?"
)
```

## Tool Definition Prompt

```python
TOOL_PROMPT = """You have access to the following tools:

{tools}

When to use each tool:
- Use {tool_name} when: {when_to_use}
- Do NOT use {tool_name} when: {when_not_to_use}

Guidelines:
{guidelines}"""

def create_tool_prompt(tools: list) -> str:
    tool_sections = []
    for tool in tools:
        tool_sections.append(f"**{tool.name}**: {tool.description}")
    
    return TOOL_PROMPT.format(tools="\n\n".join(tool_sections))
```

## Structured Output Prompt

```python
from pydantic import BaseModel

class OrderStatus(BaseModel):
    order_id: str
    status: str
    estimated_delivery: str
    tracking_number: str | None

STRUCTURED_OUTPUT_PROMPT = """Extract order information from the text below.

{context}

Return a JSON object with:
- order_id: The order identifier
- status: Current status (processing, shipped, delivered)
- estimated_delivery: Expected delivery date
- tracking_number: Shipping tracking number if available"""
```

## Few-Shot Prompting

```python
FEW_SHOT_PROMPT = """Example conversations:

Example 1:
User: What's the status of order #123?
Assistant: Let me check that for you.
[TOOL: check_order_status(order_id="123")]
Result: Your order #123 was shipped on Jan 15 and will arrive by Jan 18.
Assistant: Your order #123 was shipped on January 15th and is expected to arrive by January 18th.

Example 2:
User: I want to return item from order #456
Assistant: I can help you with that return. What is the reason for the return?
[TOOL: get_order_details(order_id="456")]
"""

def create_few_shot_prompt(examples: list[dict], current_query: str) -> str:
    prompt = FEW_SHOT_PROMPT + "\n\nCurrent conversation:\n"
    for ex in examples:
        prompt += f"User: {ex['user']}\nAssistant: {ex['assistant']}\n"
    prompt += f"User: {current_query}"
    return prompt
```

## Chain of Thought Prompting

```python
COT_PROMPT = """Think through this step by step:

1. What is the user asking for?
2. What information do I need?
3. What tools should I use?
4. What is the best response?

Question: {question}

Think Aloud:
{thoughts}

Response:"""
```

## Context-Aware Prompt

```python
def create_context_prompt(context: dict, query: str) -> str:
    sections = []
    
    if context.get("conversation_history"):
        sections.append("Previous conversation:")
        for msg in context["conversation_history"][-3:]:
            sections.append(f"- {msg['role']}: {msg['content']}")
    
    if context.get("user_preferences"):
        sections.append("User preferences:")
        for pref, val in context["user_preferences"].items():
            sections.append(f"- {pref}: {val}")
    
    if context.get("retrieved_docs"):
        sections.append("Relevant context:")
        for doc in context["retrieved_docs"][:2]:
            sections.append(f"- {doc.page_content[:200]}")
    
    return f"""Context:
{chr(10).join(sections)}

Current question: {query}"""
```

## Dynamic System Prompt

```python
class DynamicSystemPrompt:
    def __init__(self, base_prompt: str):
        self.base_prompt = base_prompt
        self.addons = []
    
    def add_capability(self, capability: str):
        self.addons.append(capability)
        return self
    
    def add_restriction(self, restriction: str):
        self.addons.append(f"Restriction: {restriction}")
        return self
    
    def build(self) -> str:
        prompt = self.base_prompt
        if self.addons:
            prompt += "\n\nAdditional instructions:\n" + "\n".join(f"- {a}" for a in self.addons)
        return prompt

system_prompt = (
    DynamicSystemPrompt("You are a helpful retail assistant.")
    .add_capability("Can check order status")
    .add_capability("Can process returns")
    .add_restriction("Never share user passwords")
    .build()
)
```

## Prompt Versioning

```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class PromptVersion:
    version: str
    prompt: str
    created_at: datetime
    description: str

PROMPT_REGISTRY = {
    "retail_assistant_v1": PromptVersion(
        version="1.0",
        prompt=SYSTEM_PROMPT,
        created_at=datetime.now(),
        description="Initial retail assistant prompt"
    )
}
```

## Quick Reference

| Template | Use |
|----------|-----|
| `ChatPromptTemplate` | Combine multiple message types |
| `SystemMessagePromptTemplate` | Agent identity |
| `HumanMessagePromptTemplate` | User input |
| `FewShotPromptTemplate` | Example-based learning |
| `StructuredOutputParser` | JSON output |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Vague instructions | Be specific about behavior |
| No tool descriptions | Explain when to use tools |
| No error handling | Include error cases in prompt |
| Inconsistent style | Use template variables |

## Dependencies

```bash
pip install langchain
```

## Next Steps

- `ollama-integration` - Use prompts with Ollama
- `langgraph-agent-setup` - Apply prompts to agent
- `agentic-debugging` - Debug prompt issues
