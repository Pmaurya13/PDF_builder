# Workflow Graph Engine

A minimal workflow/graph engine API built with FastAPI. This project implements a simplified version of a workflow orchestration system, similar to LangGraph, where you can define nodes, connect them with edges, maintain shared state, and run workflows end-to-end via APIs.

## Features

### Core Engine Capabilities

- **Nodes**: Python functions that read and modify shared state
- **State Management**: Dictionary-based state that flows between nodes
- **Edges**: Define execution flow between nodes with optional conditions
- **Conditional Branching**: Route execution based on state values
- **Looping**: Repeat nodes until conditions are met (with max iteration limits)
- **Tool Registry**: Register and call Python functions from nodes

### API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/graph/create` | POST | Create a new workflow graph |
| `/graph/run` | POST | Execute a graph with initial state |
| `/graph/state/{run_id}` | GET | Get current state of a workflow run |
| `/graph/list` | GET | List all registered graphs |
| `/graph/{graph_id}` | GET | Get details of a specific graph |
| `/graph/tools` | GET | List all registered tools |

## How to Run

### Prerequisites

- Python 3.11+
- pip or uv package manager

### Installation

```bash
# Install dependencies
pip install fastapi uvicorn pydantic

# Or using uv
uv add fastapi uvicorn pydantic
```

### Running the Server

```bash
# Run with uvicorn
uvicorn app.main:app --host 0.0.0.0 --port 5000 --reload

# Or run directly
python -m app.main
```

The API will be available at `http://localhost:5000`. Interactive docs are at `/docs`.

## Example Workflow: Code Review Mini-Agent

The project includes a pre-built Code Review workflow with these steps:

1. **Extract Functions**: Parse Python code to identify functions
2. **Check Complexity**: Calculate complexity metrics
3. **Detect Issues**: Find code smells and problems
4. **Suggest Improvements**: Generate improvement suggestions
5. **Loop until quality_score >= threshold**

### Creating the Code Review Workflow

```bash
curl -X POST http://localhost:5000/graph/create \
  -H "Content-Type: application/json" \
  -d '{
    "name": "code_review_workflow",
    "start_node": "extract",
    "end_node": "complete",
    "nodes": [
      {"name": "extract", "tool_name": "extract_functions"},
      {"name": "complexity", "tool_name": "check_complexity"},
      {"name": "detect", "tool_name": "detect_issues"},
      {"name": "suggest", "tool_name": "suggest_improvements", "config": {"threshold": 70}},
      {
        "name": "quality_check",
        "node_type": "conditional",
        "tool_name": "check_quality_threshold",
        "config": {"threshold": 70},
        "branches": [
          {"condition": "meets_threshold == True", "target_node": "complete"}
        ],
        "default_target": "extract"
      },
      {"name": "complete"}
    ],
    "edges": [
      {"from_node": "extract", "to_node": "complexity"},
      {"from_node": "complexity", "to_node": "detect"},
      {"from_node": "detect", "to_node": "suggest"},
      {"from_node": "suggest", "to_node": "quality_check"}
    ]
  }'
```

### Running the Workflow

```bash
curl -X POST http://localhost:5000/graph/run \
  -H "Content-Type: application/json" \
  -d '{
    "graph_id": "YOUR_GRAPH_ID",
    "initial_state": {
      "code": "def hello_world():\n    print(\"Hello, World!\")\n\ndef add(a, b):\n    return a + b"
    }
  }'
```

## Project Structure

```
app/
├── __init__.py
├── main.py              # FastAPI application entry point
├── api/
│   ├── __init__.py
│   └── routes.py        # API endpoint definitions
├── core/
│   ├── __init__.py
│   ├── graph_engine.py  # Core workflow engine (Graph, Node, Edge classes)
│   ├── state.py         # WorkflowState management
│   └── tool_registry.py # Tool registration system
├── models/
│   ├── __init__.py
│   └── schemas.py       # Pydantic models for API
└── workflows/
    ├── __init__.py
    └── code_review_tools.py  # Code review workflow tools
```

## Node Types

### Standard Node
Basic node that executes a tool and passes state forward.

```json
{
  "name": "my_node",
  "tool_name": "my_tool",
  "config": {}
}
```

### Conditional Node
Routes execution based on state conditions.

```json
{
  "name": "check_node",
  "node_type": "conditional",
  "branches": [
    {"condition": "score > 80", "target_node": "success"},
    {"condition": "score > 50", "target_node": "retry"}
  ],
  "default_target": "fail"
}
```

### Loop Node
Repeats execution until a condition is met.

```json
{
  "name": "loop_node",
  "node_type": "loop",
  "loop_body_start": "process",
  "loop_condition": "count < 5",
  "max_iterations": 10,
  "exit_node": "done"
}
```

## What I Would Improve With More Time

1. **Persistent Storage**: Add PostgreSQL/SQLite for storing graphs and runs
2. **WebSocket Streaming**: Real-time execution log streaming
3. **Async Execution**: Background task support for long-running workflows
4. **Parallel Nodes**: Execute independent nodes concurrently
5. **Subgraphs**: Nested workflow support
6. **Retry Logic**: Automatic retry with exponential backoff
7. **Workflow Versioning**: Track and manage graph versions
8. **Metrics & Monitoring**: Execution time tracking and performance metrics
9. **Authentication**: Secure API access with JWT tokens
10. **Visualization**: Graph visualization endpoint

## License

MIT License
