---
name: fastapi-agent-endpoints
description: Use when creating FastAPI endpoints for LangGraph agents - REST endpoints for agent invocation, WebSocket for streaming agent thoughts, and async task handling
---

# FastAPI Agent Endpoints

## Overview

Expose LangGraph agents via FastAPI with REST endpoints and WebSocket streaming for real-time agent execution display.

## When to Use

- Building API layer for agent systems
- Enabling real-time agent streaming to frontend
- Handling async agent execution
- Connecting React frontend to agent backend

## Endpoint Patterns

| Pattern | Use Case |
|---------|----------|
| `@app.post /agent/invoke` | Single agent request |
| `@app.websocket /ws/agent` | Streaming execution |
| `@app.post /agent/interrupt` | Pause running agent |
| `@app.post /agent/resume` | Resume interrupted agent |

## Basic REST Endpoint

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional
import uuid

app = FastAPI()

class AgentRequest(BaseModel):
    message: str
    thread_id: Optional[str] = None
    context: Optional[dict] = None

class AgentResponse(BaseModel):
    response: str
    thread_id: str
    steps: list[dict]

@app.post("/agent/invoke")
async def invoke_agent(request: AgentRequest):
    """Synchronous agent invocation."""
    thread_id = request.thread_id or str(uuid.uuid4())
    
    result = await agent.ainvoke(
        {"messages": [request.message]},
        config={"configurable": {"thread_id": thread_id}}
    )
    
    return AgentResponse(
        response=result["messages"][-1].content,
        thread_id=thread_id,
        steps=result.get("execution_steps", [])
    )
```

## WebSocket Streaming Endpoint

```python
from fastapi import WebSocket, WebSocketDisconnect
import json

class ConnectionManager:
    def __init__(self):
        self.active_connections: dict[str, WebSocket] = {}
    
    async def connect(self, thread_id: str, websocket: WebSocket):
        await websocket.accept()
        self.active_connections[thread_id] = websocket
    
    async def disconnect(self, thread_id: str):
        if thread_id in self.active_connections:
            del self.active_connections[thread_id]
    
    async def send_event(self, thread_id: str, event: dict):
        if thread_id in self.active_connections:
            await self.active_connections[thread_id].send_json(event)

manager = ConnectionManager()

@app.websocket("/ws/agent/{thread_id}")
async def websocket_agent(websocket: WebSocket, thread_id: str):
    await manager.connect(thread_id, websocket)
    
    try:
        async for event in agent.astream_events(
            {"messages": []},
            config={"configurable": {"thread_id": thread_id}}
        ):
            await manager.send_event(thread_id, {
                "type": event["event"],
                "data": event.get("data", {})
            })
    except WebSocketDisconnect:
        await manager.disconnect(thread_id)
```

## Streaming with SSE (Alternative to WebSocket)

```python
from fastapi import Request
from sse_starlette.sse import EventSourceResponse

@app.get("/agent/stream/{thread_id}")
async def stream_agent(thread_id: str, request: Request):
    """Server-Sent Events for agent streaming."""
    
    async def event_generator():
        async for event in agent.astream_events(
            {"messages": []},
            config={"configurable": {"thread_id": thread_id}}
        ):
            if await request.is_disconnected():
                break
            yield {
                "event": event["event"],
                "data": json.dumps(event.get("data", {}))
            }
    
    return EventSourceResponse(event_generator())
```

## Interrupt and Resume Endpoints

```python
@app.post("/agent/interrupt/{thread_id}")
async def interrupt_agent(thread_id: str):
    """Pause a running agent."""
    agent.interrupt(thread_id)
    return {"status": "interrupted", "thread_id": thread_id}

class ResumeRequest(BaseModel):
    user_input: str

@app.post("/agent/resume/{thread_id}")
async def resume_agent(thread_id: str, request: ResumeRequest):
    """Resume interrupted agent with user input."""
    result = await agent.invoke(
        Command(resume={"user_input": request.user_input}),
        config={"configurable": {"thread_id": thread_id}}
    )
    return {"response": result["messages"][-1].content}
```

## Thread-Based Conversation

```python
from langgraph.checkpoint.memory import MemorySaver

checkpointer = MemorySaver()

agent = workflow.compile(checkpointer=checkpointer)

@app.post("/agent/chat/{thread_id}")
async def chat_agent(thread_id: str, request: AgentRequest):
    """Continue conversation in existing thread."""
    result = await agent.ainvoke(
        {"messages": [HumanMessage(content=request.message)]},
        config={
            "configurable": {"thread_id": thread_id},
            "recursion_limit": 50
        }
    )
    return {"response": result["messages"][-1].content}
```

## Quick Reference

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/agent/invoke` | POST | Single request |
| `/ws/agent/{id}` | WS | Stream execution |
| `/agent/chat/{id}` | POST | Threaded chat |
| `/agent/interrupt/{id}` | POST | Pause agent |
| `/agent/resume/{id}` | POST | Resume agent |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| No thread_id | Use thread_id for conversation memory |
| Blocking calls | Use `ainvoke` for async |
| Missing checkpointer | Use MemorySaver or SQLite |
| No connection cleanup | Handle WebSocketDisconnect |

## Dependencies

```bash
pip install fastapi uvicorn sse-starlette
```

## Next Steps

- `react-streaming-display` - Consume WebSocket in React
- `agent-state-visualization` - Show execution steps
- `interactive-agent-controls` - Pause/resume from UI
