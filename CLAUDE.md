# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh && ./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install dependencies
uv sync

# Add new dependency
uv add package_name
```

### Environment Setup
Create `.env` file in root directory:
```bash
ANTHROPIC_API_KEY=your_anthropic_api_key_here
```

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) system** for course materials with a multi-layered architecture:

### Core Architecture Pattern
**Frontend → FastAPI → RAG System → AI Generator ↔ Tool System → Vector Store**

The system follows a tool-based RAG approach where Claude AI decides when to search course content rather than always performing retrieval.

### Key Architectural Components

**1. RAG System (`backend/rag_system.py`)**
- Central orchestrator that coordinates all components
- Manages the query processing pipeline
- Integrates session management, AI generation, and tool execution
- Entry point: `query()` method processes user queries

**2. Tool-Based Search Architecture (`backend/search_tools.py`)**
- Implements Anthropic's tool calling interface
- `CourseSearchTool` provides semantic search capabilities
- `ToolManager` handles tool registration and execution
- AI decides when tools are needed, not every query triggers search

**3. Dual Vector Store Design (`backend/vector_store.py`)**
- **Course Catalog Collection**: Stores course metadata and titles for semantic course name matching
- **Course Content Collection**: Stores chunked course content for detailed search
- Uses ChromaDB with sentence-transformer embeddings (`all-MiniLM-L6-v2`)

**4. Session Management (`backend/session_manager.py`)**
- Maintains conversation context with configurable history limits
- Stores user-assistant message exchanges
- Provides formatted conversation history to AI for context

**5. Document Processing Pipeline (`backend/document_processor.py`)**
- Extracts structured course information from text files
- Intelligent sentence-based text chunking with overlap
- Creates `Course`, `Lesson`, and `CourseChunk` objects

### Data Models (`backend/models.py`)

**Core Models:**
- `Course`: Contains title, instructor, course_link, and lessons list
- `Lesson`: Individual lesson with number, title, and optional link
- `CourseChunk`: Text chunks with course/lesson metadata for vector storage

### Configuration System (`backend/config.py`)

**Key Settings:**
- `CHUNK_SIZE: 800` - Text chunk size for vector storage
- `CHUNK_OVERLAP: 100` - Character overlap between chunks
- `MAX_RESULTS: 5` - Maximum search results returned
- `MAX_HISTORY: 2` - Conversation messages remembered
- `ANTHROPIC_MODEL: "claude-sonnet-4-20250514"` - AI model used

### Frontend Integration (`frontend/`)

**Single-Page Application:**
- `script.js`: Handles user interactions and API communication
- Session management on frontend coordinates with backend sessions
- Real-time course statistics loading
- Markdown rendering for AI responses with collapsible source citations

## Query Processing Flow

1. **User Input**: Frontend captures query and session ID
2. **API Endpoint**: FastAPI `/api/query` receives request
3. **Session Resolution**: Create new session or retrieve existing conversation history
4. **RAG Processing**: `RAGSystem.query()` orchestrates the pipeline
5. **AI Decision**: Claude determines if course search is needed
6. **Tool Execution**: If needed, `CourseSearchTool` performs semantic search
7. **Context Integration**: Search results combined with conversation history
8. **Response Generation**: Final answer generated with source citations
9. **Session Update**: Q&A exchange stored for future context

## Important Implementation Details

### Vector Store Collections
- **Never directly query collections** - use `VectorStore.search()` method
- Course name resolution happens automatically via semantic matching
- Filtering supports both course names and lesson numbers simultaneously

### AI Tool Integration
- Tools are **selectively used** - not every query triggers search
- System prompt in `AIGenerator` defines tool usage strategy
- `ToolManager` handles tool execution and source collection

### Session Lifecycle
- Sessions auto-created if not provided in API requests
- History limited by `MAX_HISTORY` configuration to manage context size
- Session IDs managed both frontend and backend

### Document Loading
- Startup event automatically loads documents from `docs/` folder
- Duplicate course detection prevents re-processing existing content
- Supports `.txt`, `.pdf`, and `.docx` files

### Development Notes
- ChromaDB persisted in `./chroma_db` directory
- CORS enabled for development with wildcard origins
- Static files served from `frontend/` directory
- Hot reload enabled in development mode

## Error Handling Patterns

- Vector store operations return `SearchResults` objects with error handling
- Tool execution failures gracefully degrade to error messages
- File processing errors are logged but don't stop batch operations
- API endpoints use FastAPI's automatic exception handling
- always use uv ro run the server do not use pip directly
- make sure to use uv to manage all dependencies