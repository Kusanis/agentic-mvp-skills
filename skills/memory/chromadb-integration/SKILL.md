---
name: chromadb-integration
description: Use when integrating ChromaDB vector database with a LangGraph agent - storing embeddings, creating collections, and enabling semantic search for agent memory
---

# ChromaDB Integration

## Overview

Connect ChromaDB to LangGraph agents for semantic memory storage and retrieval using embeddings.

## When to Use

- Adding persistent memory to an agent
- Enabling semantic search over documents
- Building RAG (Retrieval-Augmented Generation) systems
- Storing conversation context for later retrieval

## Setup

```bash
pip install chromadb langchain-chroma
```

## Basic ChromaDB Client

```python
import chromadb
from chromadb.config import Settings

client = chromadb.Client(Settings(
    persist_directory="./chroma_db",
    anonymized_telemetry=False
))
```

## Collection Management

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(
    model="llama3",
    base_url="http://localhost:11434/v1",
    api_key="ollama"
)

collection_name = "agent_memory"

vectorstore = Chroma(
    client=client,
    collection_name=collection_name,
    embedding_function=embeddings
)
```

## Adding Documents

```python
documents = [
    "Customer order #12345 was shipped on 2024-01-15",
    "Product SKU-A123 has 50 units in warehouse A",
    "Return policy allows 30 days for returns",
]

ids = ["doc1", "doc2", "doc3"]

vectorstore.add_texts(texts=documents, ids=ids)
```

## Similarity Search

```python
results = vectorstore.similarity_search(
    query="What happened with order 12345?",
    k=3  # Return top 3 results
)

for doc in results:
    print(f"Content: {doc.page_content}")
    print(f"Metadata: {doc.metadata}")
```

## With Metadata Filtering

```python
results = vectorstore.similarity_search(
    query="warehouse inventory",
    filter={"category": "inventory"},
    k=5
)
```

## As Retriever for LangGraph

```python
from langgraph.prebuilt import create_retriever

retriever = vectorstore.as_retriever(
    search_kwargs={"k": 3}
)

def retrieve_context(state: AgentState) -> AgentState:
    """Retrieve relevant context for current query."""
    last_message = state["messages"][-1].content
    
    docs = retriever.invoke(last_message)
    context = "\n".join([doc.page_content for doc in docs])
    
    return {"context": {"documents": docs, "summary": context}}
```

## Embedding Configuration

```python
from chromadb.api.models.Collection import EmbeddingFunction

class OllamaEmbeddings(EmbeddingFunction):
    def __call__(self, texts: list[str]) -> list[list[float]]:
        import requests
        response = requests.post(
            "http://localhost:11434/api/embeddings",
            json={"model": "llama3", "prompt": texts[0]}
        )
        return [response.json()["embedding"]]
```

## Collection for Different Memory Types

```python
memory_collections = {
    "conversations": "chat_history",
    "facts": "agent_knowledge",
    "documents": "product_catalog",
    "user_preferences": "user_memory"
}

for purpose, name in memory_collections.items():
    Chroma(
        client=client,
        collection_name=name,
        embedding_function=embeddings
    )
```

## Quick Reference

| Operation | Method |
|-----------|--------|
| Create client | `chromadb.Client()` |
| Add documents | `vectorstore.add_texts()` |
| Search | `vectorstore.similarity_search()` |
| As retriever | `vectorstore.as_retriever()` |
| Delete | `vectorstore.delete()` |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Wrong embedding dimension | Match model output dimension |
| No persist directory | Use `persist_directory` for persistence |
| Duplicate IDs | Use unique IDs per document |

## Dependencies

```bash
pip install chromadb langchain-chroma
```

## Next Steps

- `rag-context-management` - Retrieve and inject context into agent
- `agent-memory-patterns` - Design memory structures
- `langgraph-agent-setup` - Integrate memory into agent state
