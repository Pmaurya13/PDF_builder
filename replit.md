# Workflow Graph Engine

## Overview
A minimal workflow/graph engine API built with FastAPI for the AI Engineering Internship assignment. The system allows defining workflow graphs with nodes, edges, state management, conditional branching, and looping capabilities.

## Project Architecture

```
app/
├── main.py              # FastAPI application entry point
├── api/
│   └── routes.py        # API endpoint definitions
├── core/
│   ├── graph_engine.py  # Core workflow engine (Graph, Node, Edge, WorkflowRun)
│   ├── state.py         # WorkflowState management class
│   └── tool_registry.py # Singleton tool registration system
├── models/
│   └── schemas.py       # Pydantic models for API requests/responses
└── workflows/
    └── code_review_tools.py  # Code review workflow tools
```

## Key Components

### Graph Engine (app/core/graph_engine.py)
- **Node**: Base class for workflow nodes, executes tools
- **ConditionalNode**: Routes based on state conditions
- **LoopNode**: Repeats until condition met
- **Edge**: Connects nodes with optional conditions
- **Graph**: Container for nodes and edges
- **WorkflowRun**: Manages execution state and logging
- **GraphEngine**: Singleton managing all graphs and runs

### Tool Registry (app/core/tool_registry.py)
- Singleton pattern for global tool access
- `@register_tool(name)` decorator for easy registration
- Tools receive (state_dict, config) and return state updates

### Code Review Tools (app/workflows/code_review_tools.py)
- extract_functions: AST-based Python function extraction
- check_complexity: Complexity scoring
- detect_issues: Code smell detection
- suggest_improvements: Improvement suggestions with quality scoring
- check_quality_threshold: Loop termination check

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/graph/create` | POST | Create workflow graph |
| `/graph/run` | POST | Execute graph with initial state |
| `/graph/state/{run_id}` | GET | Get workflow run state |
| `/graph/list` | GET | List all graphs |
| `/graph/{graph_id}` | GET | Get graph details |
| `/graph/tools` | GET | List registered tools |

## Running the Project

```bash
uvicorn app.main:app --host 0.0.0.0 --port 5000
```

## Recent Changes

- 2025-12-10: Initial implementation
  - Created graph engine with Node, ConditionalNode, LoopNode support
  - Implemented tool registry with decorator-based registration
  - Added FastAPI endpoints for graph CRUD and execution
  - Implemented Code Review Mini-Agent workflow
  - Created comprehensive README documentation

## User Preferences

(No specific preferences recorded yet)
