---
name: interactive-agent-controls
description: Use when building React components for agent control - pause, resume, interrupt functionality and user input during agent execution
---

# Interactive Agent Controls

## Overview

Build React components that allow users to control agent execution - pausing, resuming, interrupting, and providing input during agent runs.

## When to Use

- Human-in-the-loop agent systems
- User approval workflows
- Debugging agent behavior
- Interactive agent sessions

## Control Buttons Component

```typescript
import React, { useState } from 'react';

interface AgentControlsProps {
  threadId: string;
  isRunning: boolean;
  isPaused: boolean;
  onInterrupt: () => Promise<void>;
  onResume: () => Promise<void>;
  onRestart: () => void;
}

export function AgentControls({
  threadId,
  isRunning,
  isPaused,
  onInterrupt,
  onResume,
  onRestart,
}: AgentControlsProps) {
  return (
    <div className="agent-controls">
      <button 
        onClick={onInterrupt}
        disabled={!isRunning || isPaused}
        className="btn-interrupt"
      >
        ⏸️ Pause
      </button>
      
      <button 
        onClick={onResume}
        disabled={!isPaused}
        className="btn-resume"
      >
        ▶️ Resume
      </button>
      
      <button 
        onClick={onRestart}
        disabled={isRunning && !isPaused}
        className="btn-restart"
      >
        🔄 Restart
      </button>
      
      <span className="status">
        {isRunning ? 'Running' : isPaused ? 'Paused' : 'Idle'}
      </span>
    </div>
  );
}
```

## Human Input Modal

```typescript
interface HumanInputModalProps {
  isOpen: boolean;
  prompt: string;
  context?: any;
  onSubmit: (input: string) => void;
  onCancel: () => void;
}

export function HumanInputModal({
  isOpen,
  prompt,
  context,
  onSubmit,
  onCancel,
}: HumanInputModalProps) {
  const [input, setInput] = useState('');
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay">
      <div className="modal-content">
        <h3>Human Input Required</h3>
        
        <div className="prompt">
          <p>{prompt}</p>
        </div>
        
        {context && (
          <div className="context-viewer">
            <h4>Current Context</h4>
            <pre>{JSON.stringify(context, null, 2)}</pre>
          </div>
        )}
        
        <textarea
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Enter your response..."
          rows={4}
        />
        
        <div className="modal-actions">
          <button onClick={onCancel} className="btn-cancel">
            Cancel
          </button>
          <button 
            onClick={() => onSubmit(input)} 
            disabled={!input.trim()}
            className="btn-submit"
          >
            Submit
          </button>
        </div>
      </div>
    </div>
  );
}
```

## Approval Queue

```typescript
interface ApprovalRequest {
  id: string;
  type: 'confirm' | 'choice' | 'input';
  title: string;
  description?: string;
  options?: string[];
  onApprove: () => void;
  onDeny: () => void;
  onTimeout?: () => void;
}

export function ApprovalQueue({ requests }: { requests: ApprovalRequest[] }) {
  if (requests.length === 0) return null;
  
  const current = requests[0];
  
  return (
    <div className="approval-queue">
      <div className="queue-header">
        <h3>⚠️ Approval Required</h3>
        <span className="pending-count">{requests.length} pending</span>
      </div>
      
      <div className="current-approval">
        <h4>{current.title}</h4>
        {current.description && <p>{current.description}</p>}
        
        {current.type === 'confirm' && (
          <div className="confirm-actions">
            <button onClick={current.onApprove} className="btn-approve">
              ✓ Approve
            </button>
            <button onClick={current.onDeny} className="btn-deny">
              ✗ Deny
            </button>
          </div>
        )}
        
        {current.type === 'choice' && current.options && (
          <div className="choice-actions">
            {current.options.map((option, i) => (
              <button key={i} onClick={() => current.onApprove()}>
                {option}
              </button>
            ))}
          </div>
        )}
      </div>
      
      {requests.length > 1 && (
        <div className="queue-preview">
          <h5>Pending ({requests.length - 1})</h5>
          {requests.slice(1).map((req, i) => (
            <div key={req.id} className="queue-item">
              {req.title}
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

## Control Hook

```typescript
import { useState, useCallback } from 'react';

export function useAgentControls(threadId: string) {
  const [isRunning, setIsRunning] = useState(false);
  const [isPaused, setIsPaused] = useState(false);
  const [pendingInput, setPendingInput] = useState<{
    prompt: string;
    context?: any;
  } | null>(null);
  
  const interrupt = useCallback(async () => {
    const response = await fetch(`/api/agent/interrupt/${threadId}`, {
      method: 'POST',
    });
    if (response.ok) {
      setIsRunning(false);
      setIsPaused(true);
    }
  }, [threadId]);
  
  const resume = useCallback(async () => {
    const response = await fetch(`/api/agent/resume/${threadId}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ user_input: '' }),
    });
    if (response.ok) {
      setIsPaused(false);
      setIsRunning(true);
    }
  }, [threadId]);
  
  const submitInput = useCallback(async (input: string) => {
    const response = await fetch(`/api/agent/resume/${threadId}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ user_input: input }),
    });
    if (response.ok) {
      setPendingInput(null);
      setIsPaused(false);
      setIsRunning(true);
    }
  }, [threadId]);
  
  return {
    isRunning,
    isPaused,
    pendingInput,
    interrupt,
    resume,
    submitInput,
    setIsRunning,
  };
}
```

## Full Interactive Agent Component

```typescript
export function InteractiveAgent({ threadId }: { threadId: string }) {
  const { 
    isRunning, 
    isPaused, 
    pendingInput,
    messages,
    interrupt, 
    resume, 
    submitInput 
  } = useAgentControls(threadId);
  
  return (
    <div className="interactive-agent">
      <AgentControls
        threadId={threadId}
        isRunning={isRunning}
        isPaused={isPaused}
        onInterrupt={interrupt}
        onResume={resume}
        onRestart={() => window.location.reload()}
      />
      
      <MessageChain messages={messages} />
      
      <AgentChat threadId={threadId} />
      
      <HumanInputModal
        isOpen={!!pendingInput}
        prompt={pendingInput?.prompt || ''}
        context={pendingInput?.context}
        onSubmit={submitInput}
        onCancel={interrupt}
      />
    </div>
  );
}
```

## Keyboard Shortcuts

```typescript
import { useEffect } from 'react';

export function useAgentShortcuts(controls: {
  interrupt: () => void;
  resume: () => void;
}) {
  useEffect(() => {
    const handler = (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        controls.interrupt();
      }
      if (e.key === 'Enter' && e.ctrlKey) {
        controls.resume();
      }
    };
    
    window.addEventListener('keydown', handler);
    return () => window.removeEventListener('keydown', handler);
  }, [controls]);
}
```

## Quick Reference

| Component | Purpose |
|-----------|---------|
| `AgentControls` | Pause/resume/restart buttons |
| `HumanInputModal` | Get user input during execution |
| `ApprovalQueue` | Manage approval requests |
| `useAgentControls` | Hook for control logic |
| `useAgentShortcuts` | Keyboard shortcuts |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| No loading states | Show spinner during API calls |
| Missing error handling | Show errors to user |
| Forgetting cleanup | Cancel requests on unmount |
| No timeout | Add timeout for approvals |

## Next Steps

- `react-streaming-display` - Show execution
- `agent-state-visualization` - Visualize state
- `fastapi-agent-endpoints` - Backend controls
