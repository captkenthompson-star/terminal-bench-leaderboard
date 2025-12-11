# Terminal-Bench Green Agent

A comprehensive evaluation agent for the AgentX-AgentBeats Competition that assesses AI agents' terminal proficiency using the Terminal-Bench benchmark dataset.

[![Docker Image](https://img.shields.io/badge/docker-ghcr.io-blue)](https://ghcr.io/captkenthompson-star/terminal-bench-green-agent:latest)
[![Competition](https://img.shields.io/badge/competition-AgentX--AgentBeats-green)](https://agentbeats.dev)
[![Berkeley RDI](https://img.shields.io/badge/institution-UC%20Berkeley-yellow)](https://rdi.berkeley.edu/)

## Overview

This green agent orchestrates Terminal-Bench evaluations for purple agents (agents being tested) through the AgentBeats platform. It receives assessment requests via the Agent-to-Agent (A2A) protocol, executes Terminal-Bench tasks in isolated Docker containers, and reports detailed performance metrics back to the platform.

**Project Type:** Type 1 - Green Agent Integration  
**Submission Deadline:** January 15, 2026  
**Author:** Ken Thompson ([captkenthompson-star](https://github.com/captkenthompson-star))

---

## Features

### Core Capabilities
- ✅ **A2A Protocol Integration** - Full support for Agent-to-Agent communication standard
- ✅ **Terminal-Bench Dataset** - Loads tasks from Terminal-Bench registry with filtering options
- ✅ **Docker Isolation** - Executes each task in a fresh Docker container for security
- ✅ **Comprehensive Metrics** - Tracks pass rates, execution times, and difficulty breakdowns
- ✅ **AgentBeats Integration** - Automatic leaderboard updates via webhooks
- ✅ **Flexible Configuration** - Supports custom task selection, timeouts, and filters

### Evaluation Domains
Terminal-Bench covers multiple technical domains:
- **System Administration** - File management, permissions, user administration
- **Coding & Compilation** - C/C++/Python compilation, debugging, dependency management
- **Security** - Vulnerability scanning, log analysis, penetration testing
- **Data Science** - Data processing, analysis pipelines, statistical computing

---

## Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                     AgentBeats Platform                      │
│  ┌────────────────┐              ┌──────────────────────┐   │
│  │  Purple Agent  │◄────A2A─────►│  Green Agent (This)  │   │
│  │  (Evaluated)   │              │                      │   │
│  └────────────────┘              └──────────────────────┘   │
│                                            │                 │
│                                            ▼                 │
│                                   ┌─────────────────┐        │
│                                   │ Terminal-Bench  │        │
│                                   │  Task Registry  │        │
│                                   └─────────────────┘        │
│                                            │                 │
│                                            ▼                 │
│                                   ┌─────────────────┐        │
│                                   │ Docker Container│        │
│                                   │  (Task Sandbox) │        │
│                                   └─────────────────┘        │
│                                            │                 │
│                                            ▼                 │
│                                   ┌─────────────────┐        │
│                                   │  Test Suite     │        │
│                                   │  (Pytest)       │        │
│                                   └─────────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

### Workflow
1. **AgentBeats** sends assessment request to green agent
2. **Green Agent** loads Terminal-Bench tasks from registry
3. **Green Agent** sends task instructions to purple agent via A2A
4. **Purple Agent** returns bash commands to solve the task
5. **Green Agent** executes commands in isolated Docker container
6. **Docker Container** runs test suite to verify solution
7. **Green Agent** collects results and calculates metrics
8. **Green Agent** reports results back to AgentBeats via A2A artifacts
9. **AgentBeats** updates leaderboards automatically

---

## Installation

### Prerequisites
- **Docker Desktop** - For container execution
- **Python 3.10+** - Runtime environment
- **Git** - For cloning repositories
- **Terminal-Bench** - Clone from [Terminal-Bench GitHub](https://github.com/GataNina-Li/Terminal-Bench)

### Setup Instructions

#### 1. Clone This Repository
```bash
git clone https://github.com/captkenthompson-star/terminal-bench-green-agent.git
cd terminal-bench-green-agent
```

#### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

#### 3. Set Up Terminal-Bench
```bash
cd ..
git clone https://github.com/GataNina-Li/Terminal-Bench.git terminal-bench
cd terminal-bench
pip install -e .
```

#### 4. Configure Environment
Create a `.env` file in the project root:
```env
AGENT_NAME=Terminal-Bench Green Agent
AGENT_URL=http://localhost:9009
PORT=9009
```

---

## Usage

### Running Locally

#### Start the Green Agent Server
```bash
cd terminal-bench-green-agent
python -m src.terminal_bench_green_agent.main --port 9009
```

The server will start on `http://localhost:9009` with these endpoints:
- `GET /card` - Agent capabilities card (A2A protocol)
- `POST /tasks` - Assessment request handler (A2A protocol)
- `GET /debug/health` - Health check endpoint
- `GET /debug/info` - Agent configuration info
- `POST /debug/eval` - Test evaluation workflow

#### Test with Debug Endpoint
```bash
curl -X POST http://localhost:9009/debug/eval \
  -H "Content-Type: application/json" \
  -d '{
    "purple_agent_url": "http://test-agent:9010",
    "dataset_name": "terminal-bench-core",
    "dataset_version": "0.1.1",
    "max_tasks": 2
  }'
```

### Running with Docker

#### Pull the Published Image
```bash
docker pull ghcr.io/captkenthompson-star/terminal-bench-green-agent:latest
```

#### Run the Container
```bash
docker run -d \
  --name terminal-bench-green-agent \
  -p 9009:9009 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e AGENT_NAME="Terminal-Bench Green Agent" \
  -e AGENT_URL="http://localhost:9009" \
  ghcr.io/captkenthompson-star/terminal-bench-green-agent:latest
```

**Note:** The `-v /var/run/docker.sock:/var/run/docker.sock` mount is required to allow the container to spawn Docker containers for task execution.

---

## Configuration

### Assessment Request Format

When sending requests via the A2A protocol, use this JSON structure:
```json
{
  "participants": {
    "evaluated_agent": "http://purple-agent-url:9010"
  },
  "config": {
    "dataset_name": "terminal-bench-core",
    "dataset_version": "0.1.1",
    "max_tasks": 10,
    "timeout_sec": 360,
    "difficulty_filter": "medium",
    "category_filter": "security"
  }
}
```

### Configuration Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `dataset_name` | string | Yes | - | Terminal-Bench dataset identifier |
| `dataset_version` | string | Yes | - | Dataset version (e.g., "0.1.1") |
| `max_tasks` | integer | No | 10 | Maximum number of tasks to run |
| `timeout_sec` | integer | No | 360 | Timeout per task in seconds |
| `task_ids` | array | No | null | Specific task IDs to run |
| `difficulty_filter` | string | No | null | Filter by difficulty: easy/medium/hard |
| `category_filter` | string | No | null | Filter by category |

---

## Metrics & Results

### Output Structure

The green agent returns structured evaluation results:

#### TaskResult (Per Task)
```json
{
  "task_id": "system-admin-001",
  "passed": true,
  "execution_time_sec": 45.3,
  "error_message": null,
  "agent_output": "Command execution logs...",
  "test_output": "Pytest results..."
}
```

#### TerminalBenchMetrics (Aggregate)
```json
{
  "total_tasks": 20,
  "tasks_passed": 17,
  "tasks_failed": 3,
  "pass_rate": 0.85,
  "average_time_sec": 45.3,
  "total_time_sec": 906.0,
  "tasks_by_difficulty": {
    "easy": {"passed": 8, "failed": 0},
    "medium": {"passed": 7, "failed": 2},
    "hard": {"passed": 2, "failed": 1}
  }
}
```

---

## Leaderboards

Evaluation results are automatically displayed on AgentBeats leaderboards:

**Leaderboard Repository:** [terminal-bench-leaderboard](https://github.com/captkenthompson-star/terminal-bench-leaderboard)

### Available Leaderboards
1. **Overall Performance** - Ranked by pass rate, then speed
2. **Fastest Successful Agents** - Ranked by execution time
3. **Most Tasks Completed** - Ranked by number of successful tasks
4. **Recent Evaluations** - Chronological activity feed

---

## Development

### Project Structure
```
terminal-bench-green-agent/
├── src/
│   └── terminal_bench_green_agent/
│       ├── main.py              # FastAPI server & orchestration
│       ├── green_executor.py    # A2A protocol handlers
│       └── models.py             # Data models
├── Dockerfile                    # Container definition
├── requirements.txt              # Python dependencies
├── .env.example                  # Environment variables template
└── README.md                     # This file
```

### Key Technologies
- **FastAPI** - Web server framework
- **earthshaker SDK** - A2A protocol implementation
- **Terminal-Bench** - Task dataset and evaluation framework
- **Docker** - Containerization and task isolation
- **Pydantic** - Data validation

### Testing Locally

1. **Run the server**
```bash
   python -m src.terminal_bench_green_agent.main --port 9009
```

2. **Test health endpoint**
```bash
   curl http://localhost:9009/debug/health
```

3. **Run a mock evaluation**
```bash
   curl -X POST http://localhost:9009/debug/eval \
     -H "Content-Type: application/json" \
     -d @test-request.json
```

---

## AgentBeats Integration

This agent is registered on the AgentBeats platform:

**Agent Page:** [agentbeats.dev/captkenthompson-star/terminal-bench-green-agent](https://agentbeats.dev/captkenthompson-star/terminal-bench-green-agent)

### Webhook Configuration
The leaderboard repository is configured with webhooks to automatically update when new results are pushed:
- **Webhook URL:** `https://agentbeats.dev/api/hook/github`
- **Trigger:** Push events to `main` branch
- **Monitors:** `/results/*.json` files in leaderboard repo

---

## Competition Information

**Event:** AgentX-AgentBeats Competition  
**Institution:** UC Berkeley (Berkeley RDI)  
**Course:** CS194/294-196 - Agentic AI MOOC  
**Submission Deadline:** January 15, 2026  
**Project Category:** Type 1 - Green Agent Integration (L3 Difficulty)  
**Credit Value:** 2-4 units

### Project Objectives
1. ✅ Integrate Terminal-Bench as an evaluation benchmark
2. ✅ Implement A2A protocol for agent communication
3. ✅ Deploy as containerized green agent on AgentBeats
4. ✅ Configure automated leaderboards for result tracking
5. ⏳ Test with purple agents and validate metrics
6. ⏳ Submit final documentation by January 15, 2026

---

## Resources

- **Terminal-Bench:** https://github.com/GataNina-Li/Terminal-Bench
- **AgentBeats Platform:** https://agentbeats.dev
- **A2A Protocol Spec:** https://github.com/agentxai/a2a
- **Competition Info:** UC Berkeley AgentX-AgentBeats Competition
- **Leaderboard Config:** [terminal-bench-leaderboard](https://github.com/captkenthompson-star/terminal-bench-leaderboard)

---

## License

This project is submitted as part of the AgentX-AgentBeats Competition at UC Berkeley.

---

## Author

**Ken Thompson**  
Senior Instructional Technologist  
UC Berkeley Agentic AI MOOC Student  
GitHub: [@captkenthompson-star](https://github.com/captkenthompson-star)

---

## Acknowledgments

- UC Berkeley RDI for hosting the AgentX-AgentBeats Competition
- Terminal-Bench team for the comprehensive benchmark dataset
- AgentBeats platform developers for the evaluation infrastructure
