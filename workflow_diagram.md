# RAG Legal System - Workflow Diagram

## System Overview
This is a **Retrieval-Augmented Generation (RAG)** system designed for legal document analysis, featuring semantic caching, conversation memory, and evaluation metrics.

## Architecture Components

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                RAG LEGAL SYSTEM                                 │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Flask App     │    │  Data Loader    │    │ Vector Store    │    │ Semantic Cache  │
│   (app.py)      │    │                 │    │   Manager       │    │    Manager      │
│                 │    │                 │    │                 │    │                 │
│ • /search       │    │ • PDF Loading   │    │ • ChromaDB      │    │ • ChromaDB      │
│ • /reset_memory │    │ • Text Chunking │    │ • Embeddings    │    │ • Query Cache   │
│                 │    │ • Preprocessing │    │ • Similarity    │    │ • Cosine Sim    │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │                       │
         │                       │                       │                       │
         └───────────────────────┼───────────────────────┼───────────────────────┘
                                 │                       │
                                 ▼                       ▼
                    ┌─────────────────────────────────────────────────────────┐
                    │                RAG PIPELINE                             │
                    │                                                         │
                    │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
                    │  │ LLM Client  │  │ Embeddings  │  │ Evaluation  │     │
                    │  │             │  │  Service    │  │  Manager    │     │
                    │  │ • Gemini    │  │             │  │             │     │
                    │  │ • Memory    │  │ • Google    │  │ • RAGAS     │     │
                    │  │ • Sessions  │  │   GenAI     │  │ • Metrics   │     │
                    │  └─────────────┘  └─────────────┘  └─────────────┘     │
                    └─────────────────────────────────────────────────────────┘
```

## Detailed Workflow

### 1. System Initialization
```
┌─────────────────────────────────────────────────────────────────┐
│                    SYSTEM STARTUP                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Load Configuration (config.py)                             │
│     • API Keys (Google, OpenAI)                                │
│     • File paths (PDF, DB directories)                         │
│                                                                 │
│  2. Initialize Services                                         │
│     • EmbeddingService (Google GenAI)                          │
│     • VectorStoreManager (ChromaDB)                            │
│     • SemanticCacheManager (ChromaDB)                          │
│     • LLMClient (Gemini 2.0)                                   │
│     • EvaluationManager (RAGAS)                                │
│                                                                 │
│  3. Load & Process Legal Documents                              │
│     • DataLoader reads PDF (ipc_act.pdf)                       │
│     • Split into chunks (4000 chars, 1000 overlap)             │
│     • Create embeddings                                         │
│     • Store in ChromaDB vector store                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2. Query Processing Flow
```
┌─────────────────────────────────────────────────────────────────┐
│                    QUERY PROCESSING                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User Query → Flask /search endpoint                            │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ STEP 1: Semantic Cache Check                               ││
│  │                                                             ││
│  │ • Normalize query (lowercase, trim)                         ││
│  │ • Generate query embedding                                  ││
│  │ • Search cache with cosine similarity                       ││
│  │ • Threshold: 0.85                                          ││
│  │                                                             ││
│  │ IF CACHE HIT (≥0.85 similarity):                           ││
│  │   → Return cached response immediately                      ││
│  │   → Log cache hit with timing                               ││
│  │                                                             ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ STEP 2: RAG Pipeline (Cache Miss)                          ││
│  │                                                             ││
│  │ A. Context Retrieval:                                       ││
│  │    • Get chat history for session                           ││
│  │    • Augment query with conversation context                ││
│  │    • Perform similarity search in vector store              ││
│  │    • Retrieve top-k relevant chunks (k=3)                   ││
│  │    • Extract sources and metadata                           ││
│  │                                                             ││
│  │ B. Answer Generation:                                       ││
│  │    • Create legal expert prompt                             ││
│  │    • Include retrieved context                              ││
│  │    • Send to Gemini LLM                                     ││
│  │    • Generate contextual answer                             ││
│  │                                                             ││
│  │ C. Evaluation:                                              ││
│  │    • Use RAGAS metrics                                      ││
│  │    • Calculate faithfulness score                           ││
│  │    • Calculate answer relevancy score                       ││
│  │                                                             ││
│  │ D. Cache Storage:                                           ││
│  │    • Store query-response pair                              ││
│  │    • Include evaluation metrics                             ││
│  │    • Save to semantic cache                                 ││
│  │                                                             ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3. Response Structure
```
┌─────────────────────────────────────────────────────────────────┐
│                    RESPONSE FORMAT                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  {                                                              │
│    "query": "User's original question",                         │
│    "final_answer": "Generated legal response",                  │
│    "context": "Retrieved document chunks",                      │
│    "sources": ["Page references and chunk IDs"],               │
│    "evaluation": {                                              │
│      "faithfulness": 0.95,                                     │
│      "answer_relevancy": 0.87                                  │
│    },                                                           │
│    "cached": false,                                             │
│    "time_taken": 2.34,                                         │
│    "llm_time": 1.89                                            │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Key Features

### 🧠 Conversation Memory
- **Session-based**: Each user session maintains separate conversation history
- **Context Augmentation**: Previous conversations enhance current queries
- **Memory Management**: `/reset_memory` endpoint clears session history

### ⚡ Semantic Caching
- **Intelligent Caching**: Uses embeddings and cosine similarity
- **Performance Boost**: Cached responses return in ~0.1s vs ~2s for new queries
- **Quality Preservation**: Includes evaluation metrics in cache

### 📊 Evaluation Metrics
- **Faithfulness**: How well the answer aligns with retrieved context
- **Answer Relevancy**: How relevant the answer is to the user's question
- **RAGAS Framework**: Industry-standard RAG evaluation

### 🔍 Vector Search
- **ChromaDB**: Persistent vector storage
- **Google Embeddings**: High-quality text embeddings
- **Similarity Search**: Retrieves most relevant document chunks

## Data Flow Diagram

```
User Query
    │
    ▼
┌─────────────────┐
│ Flask Endpoint  │
│   /search       │
└─────────────────┘
    │
    ▼
┌─────────────────┐    YES   ┌─────────────────┐
│ Semantic Cache  │ ────────▶│ Return Cached   │
│ Check (0.85)    │          │ Response        │
└─────────────────┘          └─────────────────┘
    │ NO
    ▼
┌─────────────────┐
│ Get Chat        │
│ History         │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ Augment Query   │
│ with Context    │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ Vector Store    │
│ Similarity      │
│ Search (k=3)    │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ Generate Answer │
│ with Gemini LLM │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ Evaluate with   │
│ RAGAS Metrics   │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ Cache Result &  │
│ Return Response │
└─────────────────┘
```

## Technology Stack

- **Backend**: Flask (Python)
- **LLM**: Google Gemini 2.0 Flash Lite
- **Embeddings**: Google GenerativeAI Embeddings
- **Vector DB**: ChromaDB
- **Document Processing**: LangChain (PyPDFLoader, RecursiveCharacterTextSplitter)
- **Evaluation**: RAGAS (faithfulness, answer_relevancy)
- **Memory**: LangChain InMemoryChatMessageHistory

## Performance Optimizations

1. **Semantic Caching**: Reduces response time by ~90% for similar queries
2. **Persistent Storage**: ChromaDB maintains embeddings across restarts
3. **Conversation Context**: Improves answer quality through session memory
4. **Chunking Strategy**: 4000 chars with 1000 overlap for optimal retrieval
5. **Evaluation Metrics**: Ensures response quality and relevance

This RAG system provides a robust foundation for legal document Q&A with intelligent caching, conversation memory, and quality evaluation.