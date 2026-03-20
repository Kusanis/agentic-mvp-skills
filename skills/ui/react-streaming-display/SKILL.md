---
name: react-streaming-display
description: Use when building React components that stream agent execution from FastAPI WebSocket - connecting to agent backends and displaying real-time streaming updates
---

# React Streaming Display

## Overview

Build React components that connect to FastAPI WebSocket endpoints and display real-time agent execution streaming.

## When to Use

- Creating UI for agent streaming
- Displaying agent thought processes
- Building real-time agent dashboards
- Enabling live agent interaction

## WebSocket Client Hook

```typescript
import { useState, useEffect, useRef, useCallback } from 'react';

interface StreamingState {
  connected: boolean;
  messages: StreamEvent[];
  error: string | null;
}

interface StreamEvent {
  type: string;
  data: any;
  timestamp: Date;
}

export function useAgentStream(threadId: string) {
  const [state, setState] = useState<StreamingState>({
    connected: false,
    messages: [],
    error: null,
  });
  
  const wsRef = useRef<WebSocket | null>(null);
  
  const connect = useCallback(() => {
    const ws = new WebSocket(`ws://localhost:8000/ws/agent/${threadId}`);
    
    ws.onopen = () => {
      setState(prev => ({ ...prev, connected: true, error: null }));
    };
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setState(prev => ({
        ...prev,
        messages: [...prev.messages, { ...data, timestamp: new Date() }],
      }));
    };
    
    ws.onerror = () => {
      setState(prev => ({ ...prev, error: 'Connection error' }));
    };
    
    ws.onclose = () => {
      setState(prev => ({ ...prev, connected: false }));
    };
    
    wsRef.current = ws;
  }, [threadId]);
  
  const disconnect = useCallback(() => {
    wsRef.current?.close();
  }, []);
  
  const sendMessage = useCallback((message: string) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify({ message }));
    }
  }, []);
  
  useEffect(() => {
    connect();
    return () => disconnect();
  }, [connect, disconnect]);
  
  return { ...state, sendMessage, reconnect: connect };
}
```

## Streaming Chat Component

```typescript
import React, { useState, useRef, useEffect } from 'react';

export function AgentChat({ threadId }: { threadId: string }) {
  const { messages, connected, sendMessage, reconnect } = useAgentStream(threadId);
  const [input, setInput] = useState('');
  const messagesEndRef = useRef<HTMLDivElement>(null);
  
  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };
  
  useEffect(scrollToBottom, [messages]);
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (input.trim() && connected) {
      sendMessage(input);
      setInput('');
    }
  };
  
  return (
    <div className="agent-chat">
      <div className="status">
        {connected ? '🟢 Connected' : '🔴 Disconnected'}
        {!connected && <button onClick={reconnect}>Reconnect</button>}
      </div>
      
      <div className="messages">
        {messages.map((msg, i) => (
          <MessageBubble key={i} event={msg} />
        ))}
        <div ref={messagesEndRef} />
      </div>
      
      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Ask the agent..."
          disabled={!connected}
        />
        <button type="submit" disabled={!connected}>Send</button>
      </form>
    </div>
  );
}
```

## Message Bubble Component

```typescript
interface MessageBubbleProps {
  event: StreamEvent;
}

function MessageBubble({ event }: MessageBubbleProps) {
  switch (event.type) {
    case 'on_chain_start':
      return <div className="message system">Agent started: {event.data?.name}</div>;
    
    case 'on_chain_stream':
      const content = event.data?.chunk?.content;
      return content ? <div className="message agent">{content}</div> : null;
    
    case 'on_tool_start':
      return <div className="message tool">Using tool: {event.data?.name}</div>;
    
    case 'on_tool_end':
      return <div className="message tool-end">Tool completed: {event.data?.name}</div>;
    
    case 'on_chain_end':
      return <div className="message system">Agent finished</div>;
    
    default:
      return null;
  }
}
```

## Agent Execution Timeline

```typescript
export function ExecutionTimeline({ events }: { events: StreamEvent[] }) {
  return (
    <div className="timeline">
      {events.map((event, index) => (
        <div key={index} className={`event event-${event.type.split('_')[1]}`}>
          <span className="timestamp">
            {event.timestamp.toLocaleTimeString()}
          </span>
          <span className="type">{formatEventType(event.type)}</span>
          {event.data?.content && (
            <span className="content">{event.data.content}</span>
          )}
        </div>
      ))}
    </div>
  );
}

function formatEventType(type: string): string {
  return type.replace(/_/g, ' ').replace('on ', '');
}
```

## Streaming Context Provider

```typescript
import React, { createContext, useContext, ReactNode } from 'react';

interface StreamingContextType {
  threadId: string;
  events: StreamEvent[];
  isConnected: boolean;
  controls: {
    pause: () => void;
    resume: () => void;
    interrupt: () => void;
  };
}

const StreamingContext = createContext<StreamingContextType | null>(null);

export function StreamingProvider({ 
  threadId, 
  children 
}: { 
  threadId: string; 
  children: ReactNode; 
}) {
  const { messages, connected } = useAgentStream(threadId);
  
  const controls = {
    pause: () => fetch(`/api/agent/interrupt/${threadId}`, { method: 'POST' }),
    resume: () => fetch(`/api/agent/resume/${threadId}`, { method: 'POST' }),
    interrupt: () => fetch(`/api/agent/interrupt/${threadId}`, { method: 'POST' }),
  };
  
  return (
    <StreamingContext.Provider value={{ threadId, events: messages, isConnected: connected, controls }}>
      {children}
    </StreamingContext.Provider>
  );
}

export const useStreaming = () => {
  const context = useContext(StreamingContext);
  if (!context) throw new Error('useStreaming must be used within StreamingProvider');
  return context;
};
```

## Quick Reference

| Hook/Component | Purpose |
|----------------|---------|
| `useAgentStream` | WebSocket connection and message handling |
| `AgentChat` | Full chat interface |
| `MessageBubble` | Display individual events |
| `ExecutionTimeline` | Timeline view of execution |
| `StreamingProvider` | Context for streaming state |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| No reconnection | Add reconnect logic |
| Memory leak | Clean up WebSocket on unmount |
| Missing loading state | Track initial connection |
| No error handling | Show error to user |

## Next Steps

- `agent-state-visualization` - Display execution steps
- `interactive-agent-controls` - Pause/resume controls
- `fastapi-agent-endpoints` - Backend WebSocket setup
