# Agentic MVP Skills

Reusable skills for building AI agent systems using **LangGraph + FastAPI + React + Ollama + ChromaDB**.

## Tech Stack

- **Agent Framework**: LangGraph
- **Backend**: FastAPI (Python)
- **Frontend**: React
- **LLM**: Ollama (Llama 3, Mistral)
- **Vector DB**: ChromaDB
- **Database**: SQLite

## Skill Categories

### Core (`skills/core/`)

| Skill | Description |
|-------|-------------|
| `langgraph-agent-setup` | Initialize agent with state schema |
| `langgraph-tool-binding` | Bind Python functions as tools |
| `langgraph-execution-flow` | Control agent decision loops |
| `fastapi-agent-endpoints` | REST + WebSocket endpoints |

### Memory (`skills/memory/`)

| Skill | Description |
|-------|-------------|
| `chromadb-integration` | Connect vector database |
| `rag-context-management` | Retrieve and inject context |
| `agent-memory-patterns` | Short/long-term memory |

### UI (`skills/ui/`)

| Skill | Description |
|-------|-------------|
| `react-streaming-display` | Stream agent to React |
| `agent-state-visualization` | Display execution steps |
| `interactive-agent-controls` | Pause/resume controls |

### LLM (`skills/llm/`)

| Skill | Description |
|-------|-------------|
| `ollama-integration` | Connect to local Ollama |
| `prompt-template-system` | Structured prompts |
| `model-switching` | Switch models dynamically |

### DevOps (`skills/devops/`)

| Skill | Description |
|-------|-------------|
| `environment-setup` | Python environment |
| `ollama-setup` | Install Ollama models |
| `deployment-checklist` | Verify deployment |

## Directory Structure

```
agentic-mvp-skills/
├── .github/                  # GitHub config (issues, PRs, funding)
├── .opencode/                # OpenCode plugin
│   ├── INSTALL.md            # Installation instructions
│   └── plugins/              # Plugin JS files
├── docs/
│   └── plans/                # Design docs and plans
├── hooks/                    # Polyglot session hooks
├── skills/                   # All skills
│   ├── core/                 # LangGraph agent skills
│   ├── memory/               # ChromaDB/RAG skills
│   ├── ui/                   # React component skills
│   ├── llm/                  # Ollama/LLM skills
│   ├── devops/               # Environment setup
│   └── using-agentic-mvp-skills/  # Entry skill
├── tests/
│   └── skill-triggering/     # Skill trigger tests
├── .gitattributes
├── .gitignore
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── LICENSE
└── README.md
```

## Installation for OpenCode

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/agentic-mvp-skills.git ~/.config/opencode/agentic-mvp-skills

# 2. Symlink the plugin
mkdir -p ~/.config/opencode/plugins
ln -s ~/.config/opencode/agentic-mvp-skills/.opencode/plugins/agentic-mvp-skills.js \
    ~/.config/opencode/plugins/agentic-mvp-skills.js

# 3. Symlink skills
mkdir -p ~/.config/opencode/skills
ln -s ~/.config/opencode/agentic-mvp-skills/skills \
    ~/.config/opencode/skills/agentic-mvp-skills

# 4. Restart OpenCode
```

## Usage

Load skills with OpenCode's native `skill` tool:

```
use skill tool to load agentic-mvp-skills/langgraph-agent-setup
```

## Companion Superpowers

Process skills for AI agent development (requires [superpowers](https://github.com/obra/superpowers)):

- `brainstorming` - Design agent systems
- `writing-plans` - Plan features
- `verification-before-completion` - Verify implementations
- `systematic-debugging` - Debug issues
- `test-driven-development` - Test agent code
- `requesting-code-review` - Review code
- `finishing-a-development-branch` - Complete work
- `executing-plans` - Execute plans
- `dispatching-parallel-agents` - Parallel work

## Workflow

```
┌─────────────┐     ┌─────────────────────────┐     ┌──────────────┐
│  Design     │────▶│  Implementation Skills  │────▶│  Verify      │
│  brainstorming      │  core/memory/ui/llm/devops │     │  deployment  │
└─────────────┘     └─────────────────────────┘     └──────────────┘
```

## Quick Start

```bash
# Install dependencies
pip install -r requirements.txt
ollama pull llama3

# Start services
ollama serve
uvicorn app:app --reload
```

## Contributing

See [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md).

## License

MIT - see [LICENSE](./LICENSE).
