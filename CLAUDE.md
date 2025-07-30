# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Quality Checks and Testing
- `make test` - Run tests using pytest
- `make test-coverage` - Run tests with coverage report
- `make check` - Run format and mypy commands (recommended before commits)
- `make format` - Format code with black and ruff
- `make mypy` - Run mypy for type checking
- `make black` - Check code format with black
- `make ruff` - Check code with ruff

### Running the Application
- `make run` - Run the application using poetry
- `make dev` - Run in development mode with auto-reload on port 8001
- `make dev-windows` - Run in development mode on Windows
- `poetry run python -m private_gpt` - Alternative way to run the application

### Utility Commands
- `make setup` - Setup the application
- `make ingest <path>` - Ingest documents from specified path
- `make wipe` - Wipe all data
- `make api-docs` - Generate API documentation

### Docker Development
- `docker-compose up -d ollama-cpu` - Start Ollama CPU service
- `docker-compose up -d private-gpt-ollama` - Start PrivateGPT with Ollama integration
- `docker-compose exec ollama-cpu ollama pull <model>` - Pull models into Ollama
- `docker-compose ps` - Check running services

## Architecture Overview

PrivateGPT is a production-ready RAG (Retrieval Augmented Generation) application built with FastAPI and LlamaIndex. It provides API building blocks for creating private, context-aware AI applications that follow and extend the OpenAI API standard with support for both normal and streaming responses.

### Core Components Structure
```
private_gpt/
├── components/          # Component implementations providing abstractions
│   ├── embedding/       # Embedding component (Ollama, OpenAI, HuggingFace, etc.)
│   ├── llm/            # LLM component (Ollama, LlamaCPP, OpenAI, etc.)
│   ├── vector_store/   # Vector store implementations (Qdrant, Chroma, etc.)
│   ├── node_store/     # Node store component
│   └── ingest/         # Document ingestion component
├── server/             # FastAPI routers and services
│   ├── chat/           # Chat endpoints and RAG pipeline
│   ├── completions/    # OpenAI-compatible completions API
│   ├── embeddings/     # Embeddings generation API
│   ├── chunks/         # Document chunks retrieval API
│   └── ingest/         # Document ingestion API
├── settings/           # Configuration and settings management
└── ui/                 # Gradio UI implementation
```

### Key Architecture Principles
- **Dependency Injection**: Uses `injector` library to decouple components
- **LlamaIndex Abstractions**: Built on LlamaIndex's `LLM`, `BaseEmbedding`, `VectorStore` abstractions
- **Component-based Design**: Each component provides implementations for specific abstractions
- **OpenAI API Compatibility**: Follows OpenAI API schema for standard endpoints, compatible with existing OpenAI-based tools
- **Privacy-First Design**: Ensures data never leaves execution environment, supporting both local and private cloud deployments
- **Streaming Support**: Full support for streaming responses in addition to standard request/response patterns

### Configuration System
- **Profile-based Configuration**: Uses YAML files (settings-*.yaml) for different deployment modes
- **Environment Variables**: Supports environment variable overrides with ${VAR:default} syntax
- **Multiple Modes**: Supports local, ollama, openai, docker, sagemaker, etc.

## Common Development Patterns

### Adding New LLM Provider
1. Create implementation in `private_gpt/components/llm/`
2. Update settings schema if needed
3. Add configuration in appropriate settings-*.yaml file
4. Register component in dependency injection system

### Adding New Vector Store
1. Implement in `private_gpt/components/vector_store/`
2. Add optional dependency in pyproject.toml
3. Update settings configuration
4. Add to component factory

### Settings Configuration
- Main settings file: `settings.yaml`
- Profile-specific overrides: `settings-{profile}.yaml`
- Docker mode uses `settings-docker.yaml` with container-friendly defaults
- Environment variables take precedence: `PGPT_PROFILES`, `PGPT_MODE`, etc.

### Docker Services
- `private-gpt-ollama`: Main application with Ollama integration
- `ollama-cpu/ollama-cuda`: Ollama model serving
- `ollama`: Traefik proxy for Ollama (when using proxy setup)
- Service names are used for inter-container communication (not localhost)

### Testing
- Tests located in `tests/` directory
- Uses pytest with async support
- Fixtures available in `tests/fixtures/`
- Run specific test: `poetry run pytest tests/path/to/test.py::test_name`

## Important Configuration Notes

### Ollama Integration
- For Docker: Use service names like `http://ollama-cpu:11434` not `localhost:11434`
- Models must be pulled before use: `docker-compose exec ollama-cpu ollama pull <model>`
- Default models: `llama3.1` (LLM), `nomic-embed-text` (embeddings)

### Development Workflow
1. Always run `make check` before committing
2. Use profile-based configuration for different environments
3. Ingest documents using `make ingest <path>` or the API
4. UI available at http://localhost:8001 when running in dev mode

### Deployment Options
- **Local Development**: Cost-free local setup using `make dev`
- **Docker Compose**: Multi-container setup with Ollama integration
- **Enterprise/Production**: Supports on-premise (data center, bare metal) and private cloud (AWS, GCP, Azure) deployments
- **OpenAI Tool Compatibility**: Can be used as drop-in replacement for OpenAI API in existing tools and workflows