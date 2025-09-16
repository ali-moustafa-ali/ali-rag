# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Development Commands

### Environment Setup
```bash
# Create conda environment
conda create -n mini-rag python=3.10
conda activate mini-rag

# Install dependencies (from src directory)
cd src
pip install -r requirements.txt

# Setup environment variables
cp .env.example .env
# Edit .env file with your API keys (OPENAI_API_KEY, COHERE_API_KEY)
```

### Database Operations
```bash
# Run database migrations (from src directory)
alembic upgrade head

# Note: Database setup requires PostgreSQL with pgvector extension
# Use docker-compose for easy setup (see Docker section)
```

### Docker Development
```bash
# Start all services (from docker directory)
cd docker
cp .env.example .env  # Edit with your credentials
sudo docker compose up -d

# Access services:
# - FastAPI: http://localhost:8000
# - Flower Dashboard: http://localhost:5555
# - Grafana: http://localhost:3000
# - Prometheus: http://localhost:9090
```

### Local Development (FastAPI)
```bash
# Run FastAPI server (from src directory)
uvicorn main:app --reload --host 0.0.0.0 --port 5000
```

### Celery Operations (Local Development)
```bash
# Run Celery worker (from src directory)
python -m celery -A celery_app worker --queues=default,file_processing,data_indexing --loglevel=info

# Run Beat scheduler (separate terminal)
python -m celery -A celery_app beat --loglevel=info

# Run Flower dashboard (separate terminal)
python -m celery -A celery_app flower --conf=flowerconfig.py
```

### Testing File Processing
Use the provided POSTMAN collection at `/assets/mini-rag-app.postman_collection.json` for API testing.

## Architecture Overview

### Core Components
- **FastAPI Application**: Main web server handling HTTP requests
- **Celery Task Queue**: Asynchronous background processing for file operations and data indexing
- **PostgreSQL + pgvector**: Primary database with vector search capabilities
- **Qdrant**: Alternative vector database option (configurable)
- **Redis**: Celery result backend and caching
- **RabbitMQ**: Message broker for Celery tasks

### Provider Factory Pattern
The system uses factory patterns for pluggable backends:
- **LLM Providers**: OpenAI, Cohere (configurable via `GENERATION_BACKEND` and `EMBEDDING_BACKEND`)
- **Vector DB Providers**: Qdrant, pgvector (configurable via `VECTOR_DB_BACKEND`)

### Key Processing Flow
1. Files uploaded via FastAPI routes → stored as Assets
2. Background tasks process files into chunks → stored as DataChunks
3. Chunks get embedded and indexed in vector database
4. Search queries use semantic similarity to retrieve relevant chunks
5. Retrieved chunks augment LLM prompts for question answering

### Task Queues
- `file_processing`: File content processing and chunking
- `data_indexing`: Vector embedding and database indexing
- `default`: General maintenance tasks

### Database Models
- **Project**: Container for related files and data
- **Asset**: Individual files uploaded to the system
- **DataChunk**: Text chunks extracted from files with metadata
- **CeleryTaskExecution**: Idempotency management for background tasks

### Configuration System
Environment-based configuration via Pydantic Settings:
- LLM API keys and model configurations
- Database connection strings
- Celery broker and backend URLs
- File processing parameters (chunk size, overlap)

### Monitoring Stack
- **Prometheus**: Metrics collection
- **Grafana**: Visualization dashboards
- **Flower**: Celery task monitoring
- Custom metrics exported via `starlette-exporter`

## Important Project Context

This is an educational RAG (Retrieval-Augmented Generation) implementation with Arabic YouTube course tutorials. Each major version corresponds to a specific tutorial branch.

### Multi-language Support
- Primary language: Arabic (`PRIMARY_LANG = "ar"`)
- Fallback language: English (`DEFAULT_LANG = "en"`)
- Template system supports localized responses

### File Processing Pipeline
The system supports PDF and plain text files with configurable chunk sizes and overlap for optimal semantic search performance.

### Idempotency Management
Background tasks include sophisticated idempotency checking to prevent duplicate processing and handle task failures gracefully.

### Development vs Production Modes
- Development: Run FastAPI and Celery services manually
- Production: Use Docker Compose with full monitoring stack

When working with this codebase, always ensure environment variables are properly configured, especially API keys for OpenAI/Cohere and database credentials.