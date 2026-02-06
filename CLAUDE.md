# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Important:** Always use `uv` to manage dependencies and run the server. Do not use `pip` directly.

## Running the Application

**Install dependencies:**
```bash
uv sync
```

**Start the server:**
```bash
cd backend && uv run uvicorn app:app --reload --port 8000
```

Or use the shell script (requires Git Bash on Windows):
```bash
./run.sh
```

**Access points:**
- Web Interface: http://localhost:8000
- API Docs: http://localhost:8000/docs

**Environment setup:**
Create a `.env` file in the root with `ANTHROPIC_API_KEY=your-key-here`

## Architecture

This is a RAG (Retrieval-Augmented Generation) chatbot for querying course materials.

### Request Flow
1. User query arrives at FastAPI endpoint (`/api/query`)
2. `RAGSystem` orchestrates the response:
   - Gets conversation history from `SessionManager`
   - Passes query to `AIGenerator` with registered tools
3. Claude (via `AIGenerator`) decides whether to use the `search_course_content` tool
4. If tool is used, `CourseSearchTool` queries `VectorStore` (ChromaDB)
5. `VectorStore.search()` handles course name resolution and content retrieval
6. Results flow back through Claude for final response generation

### Core Components (all in `backend/`)

- **RAGSystem** (`rag_system.py`): Main orchestrator that wires together all components
- **VectorStore** (`vector_store.py`): ChromaDB wrapper with two collections:
  - `course_catalog`: Course metadata for semantic course name matching
  - `course_content`: Chunked course text for content search
- **AIGenerator** (`ai_generator.py`): Anthropic API client with tool execution loop
- **DocumentProcessor** (`document_processor.py`): Parses course documents and creates chunks
- **ToolManager/CourseSearchTool** (`search_tools.py`): Tool abstraction for Claude's tool use

### Data Models (`models.py`)
- `Course`: Title, link, instructor, lessons list
- `Lesson`: Number, title, link
- `CourseChunk`: Content chunk with course/lesson metadata for vector storage

### Document Format
Course documents in `docs/` follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [name]

Lesson 1: [title]
Lesson Link: [url]
[content...]

Lesson 2: [title]
...
```

### Configuration (`config.py`)
Key settings: `CHUNK_SIZE=800`, `CHUNK_OVERLAP=100`, `MAX_RESULTS=5`, `MAX_HISTORY=2`

## Dependencies

Always use `uv` to manage all dependencies. Do not use `pip` directly.

- Install dependencies: `uv sync`
- Add a dependency: `uv add <package>`
- Remove a dependency: `uv remove <package>`

## API Endpoints

- `POST /api/query` - Query course materials (accepts `query` and optional `session_id`)
- `GET /api/courses` - Get course statistics
