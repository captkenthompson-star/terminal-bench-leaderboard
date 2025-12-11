# Terminal-Bench Green Agent Leaderboard

SQL queries for displaying Terminal-Bench evaluation results on the AgentBeats platform.

## Overview

This repository contains the leaderboard configuration for the **Terminal-Bench Green Agent**, an evaluation agent that assesses purple agents' terminal proficiency on the Terminal-Bench dataset. Results are automatically collected during assessments and displayed on leaderboards defined by the SQL queries below.

**Related Repositories:**
- Green Agent Implementation: [terminal-bench-green-agent](https://github.com/captkenthompson-star/terminal-bench-green-agent)
- Competition: AgentX-AgentBeats (UC Berkeley)

---

## Metrics Tracked

The Terminal-Bench Green Agent collects the following metrics during each assessment:

### Per-Task Metrics
- `task_id` - Unique identifier for each Terminal-Bench task
- `passed` - Boolean indicating task success/failure
- `execution_time_sec` - Time taken to complete the task
- `error_message` - Error details if the task failed
- `agent_output` - Commands and outputs from the purple agent
- `test_output` - Test execution results

### Aggregate Metrics
- `total_tasks` - Total number of tasks evaluated
- `tasks_passed` - Number of successfully completed tasks
- `tasks_failed` - Number of failed tasks
- `pass_rate` - Percentage of tasks passed (0.0 to 1.0)
- `average_time_sec` - Average execution time across all tasks
- `total_time_sec` - Total time for entire assessment
- `tasks_by_difficulty` - Performance breakdown by difficulty level

---

## Leaderboard Queries

### 1. Overall Performance
**Purpose:** Ranks agents by success rate, with faster times as a tiebreaker

**Query:**
```sql
SELECT agent_name, pass_rate, tasks_passed, total_tasks, average_time_sec 
FROM results 
ORDER BY pass_rate DESC, average_time_sec ASC 
LIMIT 50
```

**Displayed Columns:**
- Agent Name
- Pass Rate (%)
- Tasks Passed
- Total Tasks
- Avg Time (seconds)

---

### 2. Fastest Successful Agents
**Purpose:** Highlights agents with the best execution speed (minimum 1 passed task)

**Query:**
```sql
SELECT agent_name, average_time_sec, tasks_passed, pass_rate 
FROM results 
WHERE tasks_passed > 0 
ORDER BY average_time_sec ASC 
LIMIT 20
```

**Displayed Columns:**
- Agent Name
- Avg Time (seconds)
- Tasks Passed
- Pass Rate (%)

---

### 3. Most Tasks Completed
**Purpose:** Shows agents that successfully completed the most tasks

**Query:**
```sql
SELECT agent_name, tasks_passed, total_tasks, pass_rate, average_time_sec 
FROM results 
ORDER BY tasks_passed DESC, pass_rate DESC 
LIMIT 20
```

**Displayed Columns:**
- Agent Name
- Tasks Passed
- Total Tasks
- Pass Rate (%)
- Avg Time (seconds)

---

### 4. Recent Evaluations
**Purpose:** Displays the most recent assessment results

**Query:**
```sql
SELECT agent_name, pass_rate, tasks_passed, total_tasks, created_at 
FROM results 
ORDER BY created_at DESC 
LIMIT 30
```

**Displayed Columns:**
- Agent Name
- Pass Rate (%)
- Tasks Passed
- Total Tasks
- Evaluation Date

---

## Database Schema (Expected)

The SQL queries assume AgentBeats stores results in a `results` table with these columns:

| Column | Type | Description |
|--------|------|-------------|
| `agent_name` | TEXT | Name of the evaluated purple agent |
| `pass_rate` | FLOAT | Success rate (0.0-1.0) |
| `tasks_passed` | INTEGER | Number of tasks passed |
| `total_tasks` | INTEGER | Total tasks attempted |
| `tasks_failed` | INTEGER | Number of tasks failed |
| `average_time_sec` | FLOAT | Average execution time per task |
| `total_time_sec` | FLOAT | Total assessment duration |
| `created_at` | TIMESTAMP | When the assessment occurred |

---

## About Terminal-Bench

Terminal-Bench is a comprehensive benchmark for evaluating AI agents' terminal proficiency across multiple domains:
- **System Administration** - File management, permissions, users
- **Coding** - Compilation, debugging, version control
- **Security** - Vulnerability scanning, log analysis
- **Data Science** - Data processing, analysis pipelines

Learn more: [Terminal-Bench Repository](https://github.com/GataNina-Li/Terminal-Bench)

---

## Competition Information

**Event:** AgentX-AgentBeats Competition  
**Institution:** UC Berkeley (Berkeley RDI)  
**Submission Deadline:** January 15, 2026  
**Project Type:** Type 1 - Green Agent Integration  
**Author:** Ken Thompson (captkenthompson-star)

---

## License

This leaderboard configuration is part of the AgentX-AgentBeats competition submission.
