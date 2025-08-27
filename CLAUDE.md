# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start with shell script
chmod +x run.sh
./run.sh

# Manual start 
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install dependencies
uv sync

# Add new dependency
uv add package-name
```

### Environment Setup
- Copy `.env.example` to `.env` and add your `ANTHROPIC_API_KEY`
- The application requires Python 3.13+ and the `uv` package manager

## Architecture Overview

This is a Retrieval-Augmented Generation (RAG) system with a FastAPI backend and vanilla HTML/JS frontend.

### Core Components

**RAGSystem (`backend/rag_system.py`)**
- Main orchestrator that coordinates all other components
- Manages document processing, vector storage, AI generation, and session management
- Uses tool-based search architecture with CourseSearchTool

**VectorStore (`backend/vector_store.py`)**
- ChromaDB-based vector storage for course content and metadata
- Handles semantic search with embeddings using sentence-transformers
- Stores both course metadata and content chunks separately

**DocumentProcessor (`backend/document_processor.py`)**
- Processes course documents (.pdf, .docx, .txt) into structured Course and CourseChunk objects
- Handles text chunking with configurable size (800 chars) and overlap (100 chars)

**AIGenerator (`backend/ai_generator.py`)**
- Interfaces with Anthropic's Claude API (claude-sonnet-4-20250514)
- Supports tool calling for semantic search integration
- Manages conversation context and response generation

**SessionManager (`backend/session_manager.py`)**
- Maintains conversation history with configurable memory (2 messages by default)
- Enables multi-turn conversations with context

### Data Flow

1. Documents in `/docs` are processed into Course objects and CourseChunk objects
2. Course metadata and chunks are stored in separate ChromaDB collections
3. User queries trigger tool-based search through the CourseSearchTool
4. Retrieved context is passed to Claude for response generation
5. Responses include both answers and source references

### API Structure

- **POST /api/query**: Main query endpoint accepting QueryRequest with optional session_id
- **GET /api/courses**: Returns course statistics and available course titles
- **Startup Event**: Auto-loads documents from `/docs` directory on server start

### Configuration

All settings are centralized in `backend/config.py`:
- Anthropic API model and key configuration
- Embedding model (all-MiniLM-L6-v2)
- Chunk processing parameters
- ChromaDB storage location (`./chroma_db`)

### Frontend Integration

- Static frontend files served from `/frontend` directory
- Web interface available at `http://localhost:8000`
- API documentation at `http://localhost:8000/docs`
- always use uv to run the server do not use pip directly