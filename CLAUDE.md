# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DeerFlow is a multi-agent AI system built with LangGraph that coordinates specialized agents for research, planning, coding, and reporting tasks. The system consists of a Python backend with FastAPI server and a Next.js web frontend.

## Development Commands

### Backend (Python)
```bash
# Install dependencies
make install-dev

# Run development server
make serve
uv run server.py --reload

# Run tests
make test
uv run pytest tests/

# Run with coverage
make coverage
uv run pytest --cov=src tests/ --cov-report=term-missing --cov-report=xml

# Format code
make format
uv run black --preview .

# Lint code
make lint
uv run black --check .

# Run LangGraph dev server
make langgraph-dev
uvx --refresh --from "langgraph-cli[inmem]" --with-editable . --python 3.12 langgraph dev --allow-blocking
```

### Frontend (Next.js)
```bash
# Navigate to web directory
cd web

# Install dependencies
pnpm install

# Run development server
pnpm dev

# Build for production
pnpm build

# Type check
pnpm typecheck
tsc --noEmit

# Lint and type check
pnpm check
next lint && tsc --noEmit

# Format code
pnpm format:write
prettier --write "**/*.{ts,tsx,js,jsx,mdx}" --cache
```

## Architecture

### Backend Structure
- **Core Workflow**: `src/workflow.py` contains the main `run_agent_workflow_async` function that orchestrates the agent system
- **Graph Builder**: `src/graph/builder.py` defines the LangGraph state machine with nodes for different agent types
- **Agent Types**: Specialized agents in `src/agents/` include coordinator, planner, researcher, coder, and reporter
- **Configuration**: `src/config/` manages agent configurations and prompts
- **Tools**: `src/tools/` provides specialized tools for web crawling, search, TTS, and MCP integration
- **Server**: `src/server/` contains FastAPI application with chat, MCP, and RAG endpoints

### Frontend Structure
- **Core Components**: `web/src/core/` contains API hooks, state management (Zustand), and utility functions
- **Chat Interface**: `web/src/app/chat/` implements the main conversation interface with message streaming
- **Agent Visualization**: `web/src/app/landing/` provides multi-agent workflow visualization
- **UI Components**: `web/src/components/` includes reusable UI components built with Radix UI and Tailwind CSS

### Key Agent Types
1. **Planner**: Creates and refines execution plans
2. **Researcher**: Performs web searches and data collection
3. **Coder**: Executes code and handles technical tasks
4. **Reporter**: Synthesizes results into structured reports
5. **Coordinator**: Manages agent coordination and workflow progression

### Configuration
- Main configuration file: `conf.yaml` (copy from `conf.yaml.example`)
- Environment variables for API keys and model settings
- LLM configuration supports multiple providers (OpenAI, DeepSeek, custom endpoints)
- Optional reasoning model for enhanced planning capabilities

## Important Implementation Details

### Workflow Execution
- The main workflow function accepts parameters for `max_plan_iterations`, `max_step_num`, and `enable_background_investigation`
- Background investigation performs web searches before planning to enhance context
- The system uses LangGraph's state machine pattern with conditional routing between nodes
- MCP (Model Context Protocol) integration allows for external tool connections

### State Management
- Centralized state object passed between agents contains plan, results, and context
- Streaming responses via SSE (Server-Sent Events) for real-time updates
- Frontend uses Zustand for state management with persistent settings storage

### Testing
- Backend tests use pytest with coverage reporting
- Frontend tests use Next.js built-in testing tools
- Integration tests cover agent workflows and API endpoints
- Test coverage minimum threshold set to 25%

## Development Setup

1. Copy `conf.yaml.example` to `conf.yaml` and configure API keys
2. Install Python dependencies: `make install-dev`
3. Install frontend dependencies: `cd web && pnpm install`
4. Start backend server: `make serve`
5. Start frontend: `cd web && pnpm dev`

## File Structure Conventions

- Python code formatted with Black (88 character line length)
- TypeScript/React code formatted with Prettier
- Component files use PascalCase for exports
- Utility functions use camelCase
- Test files follow `test_*.py` and `*.test.*` patterns