---
name: agent-state-visualization
description: Use when building React components to visualize LangGraph agent state - showing current step, tool usage, reasoning chains, and execution history
---

# Agent State Visualization

## Overview

Build React components that visualize agent state, showing execution steps, tool calls, reasoning chains, and decision points.

## When to Use

- Dashboard for agent monitoring
- Debugging agent execution
- Showing users what the agent is doing
- Building trust through transparency

## State Visualization Types

| View | Use Case |
|------|----------|
| Step list | Sequential execution |
| Tree view | Branching decisions |
| Graph | Complex state flow |
| Timeline | Event history |

## Agent State Display Component

```typescript
import React from 'react';

interface AgentState {
  current_step: string;
  messages: any[];
  tools_used: string[];
  context: Record<string, any>;
  iteration_count: number;
}

export function AgentStatePanel({ state }: { state: AgentState }) {
  return (
    <div className="agent-state-panel">
      <div className="state-header">
        <h3>Agent State</h3>
        <span className="iteration">Iteration: {state.iteration_count}</span>
      </div>
      
      <div className="state-section">
        <h4>Current Step</h4>
        <StepIndicator step={state.current_step} />
      </div>
      
      <div className="state-section">
        <h4>Context</h4>
        <ContextViewer context={state.context} />
      </div>
      
      <div className="state-section">
        <h4>Tools Used ({state.tools_used.length})</h4>
        <ToolList tools={state.tools_used} />
      </div>
    </div>
  );
}
```

## Step Indicator

```typescript
const STEPS = [
  { id: 'idle', label: 'Idle' },
  { id: 'reasoning', label: 'Reasoning' },
  { id: 'using_tool', label: 'Using Tool' },
  { id: 'responding', label: 'Responding' },
  { id: 'complete', label: 'Complete' },
];

function StepIndicator({ step }: { step: string }) {
  const currentIndex = STEPS.findIndex(s => s.id === step);
  
  return (
    <div className="step-indicator">
      {STEPS.map((s, i) => (
        <div 
          key={s.id}
          className={`step ${i <= currentIndex ? 'active' : ''} ${i === currentIndex ? 'current' : ''}`}
        >
          <div className="step-dot" />
          <span className="step-label">{s.label}</span>
        </div>
      ))}
    </div>
  );
}
```

## Message Chain Viewer

```typescript
interface Message {
  role: 'user' | 'assistant' | 'system';
  content: string;
  tool_calls?: ToolCall[];
  tool_results?: ToolResult[];
}

export function MessageChain({ messages }: { messages: Message[] }) {
  return (
    <div className="message-chain">
      {messages.map((msg, i) => (
        <div key={i} className={`message message-${msg.role}`}>
          <div className="message-header">
            <span className="role">{msg.role}</span>
          </div>
          <div className="message-content">{msg.content}</div>
          
          {msg.tool_calls && (
            <div className="tool-calls">
              <h5>Tool Calls</h5>
              {msg.tool_calls.map((tc, j) => (
                <div key={j} className="tool-call">
                  <span className="tool-name">{tc.name}</span>
                  <pre>{JSON.stringify(tc.args, null, 2)}</pre>
                </div>
              ))}
            </div>
          )}
          
          {msg.tool_results && (
            <div className="tool-results">
              <h5>Results</h5>
              {msg.tool_results.map((tr, j) => (
                <div key={j} className="tool-result">
                  <span className="tool-name">{tr.name}</span>
                  <span className="result-content">{tr.result}</span>
                </div>
              ))}
            </div>
          )}
        </div>
      ))}
    </div>
  );
}
```

## Tool Execution Cards

```typescript
interface ToolExecution {
  name: string;
  args: Record<string, any>;
  result?: any;
  status: 'pending' | 'running' | 'complete' | 'error';
  duration?: number;
}

export function ToolExecutionCard({ tool }: { tool: ToolExecution }) {
  const statusIcon = {
    pending: '⏳',
    running: '🔄',
    complete: '✅',
    error: '❌',
  }[tool.status];
  
  return (
    <div className={`tool-card tool-${tool.status}`}>
      <div className="tool-header">
        <span className="status-icon">{statusIcon}</span>
        <span className="tool-name">{tool.name}</span>
        {tool.duration && <span className="duration">{tool.duration}ms</span>}
      </div>
      
      <div className="tool-args">
        <h5>Arguments</h5>
        <pre>{JSON.stringify(tool.args, null, 2)}</pre>
      </div>
      
      {tool.result && (
        <div className="tool-result">
          <h5>Result</h5>
          <pre>{JSON.stringify(tool.result, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

## Reasoning Chain Display

```typescript
interface ReasoningStep {
  thought: string;
  action?: string;
  observation?: string;
}

export function ReasoningChain({ steps }: { steps: ReasoningStep[] }) {
  return (
    <div className="reasoning-chain">
      <h4>Reasoning Chain</h4>
      {steps.map((step, i) => (
        <div key={i} className="reasoning-step">
          <div className="step-number">{i + 1}</div>
          <div className="step-content">
            <div className="thought">
              <span className="label">Thought:</span>
              <span>{step.thought}</span>
            </div>
            {step.action && (
              <div className="action">
                <span className="label">Action:</span>
                <span>{step.action}</span>
              </div>
            )}
            {step.observation && (
              <div className="observation">
                <span className="label">Observation:</span>
                <span>{step.observation}</span>
              </div>
            )}
          </div>
        </div>
      ))}
    </div>
  );
}
```

## Execution Graph View

```typescript
interface GraphNode {
  id: string;
  type: 'agent' | 'tool' | 'condition' | 'end';
  label: string;
  status?: 'pending' | 'active' | 'complete' | 'error';
}

export function ExecutionGraph({ 
  nodes, 
  currentNodeId 
}: { 
  nodes: GraphNode[];
  currentNodeId: string;
}) {
  return (
    <div className="execution-graph">
      {nodes.map((node) => (
        <div 
          key={node.id}
          className={`graph-node ${node.type} ${node.id === currentNodeId ? 'current' : ''}`}
        >
          <div className="node-icon">{getNodeIcon(node.type)}</div>
          <div className="node-label">{node.label}</div>
          {node.id === currentNodeId && <div className="pulse" />}
        </div>
      ))}
    </div>
  );
}

function getNodeIcon(type: string): string {
  return { agent: '🤖', tool: '🔧', condition: '❓', end: '🏁' }[type] || '⭕';
}
```

## Quick Reference

| Component | Purpose |
|-----------|---------|
| `AgentStatePanel` | Overall state display |
| `StepIndicator` | Progress through steps |
| `MessageChain` | Conversation history |
| `ToolExecutionCard` | Individual tool calls |
| `ReasoningChain` | Agent thoughts |
| `ExecutionGraph` | Visual graph view |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Too much information | Collapse older steps |
| No state updates | Subscribe to WebSocket |
| Poor hierarchy | Group by phase |
| Missing error state | Show error clearly |

## Next Steps

- `react-streaming-display` - Connect to WebSocket
- `interactive-agent-controls` - Add pause/resume
- `fastapi-agent-endpoints` - Get state from backend
