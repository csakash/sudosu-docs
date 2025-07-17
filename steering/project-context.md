---
inclusion: always
---

# AI Terminal Copilot Project Context

## Project Overview

This is **SudoSu**, an AI-powered terminal copilot that provides Cursor/Kiro-like experiences specifically for DevOps and terminal operations. The system combines sophisticated multi-agent AI workflows with real-time terminal interaction to understand user requests, create execution plans, and autonomously execute terminal commands.

## Technology Stack

### Frontend
- **Electron 28.0.0**: Desktop application framework
- **xterm.js 5.3.0**: Terminal emulation
- **Vanilla JavaScript**: UI logic and WebSocket communication
- **CSS3**: Styling and animations

### Backend
- **FastAPI 0.104.1**: Web framework and API server
- **Python 3.8+**: Core backend language
- **Google Gemini 2.0-flash**: LLM for multi-agent system
- **ptyprocess 0.7.0**: Pseudo-terminal integration
- **WebSockets**: Real-time communication
- **Pydantic**: Data validation and serialization

## Architecture Principles

### Multi-Agent System
The system implements a 4-phase agent workflow:
1. **Understanding Agent**: Analyzes user requests and breaks them into objectives
2. **Planning Agent**: Creates step-by-step execution plans with terminal commands
3. **Execution Agent**: Executes commands with monitoring and error recovery
4. **Summarization Agent**: Generates human-friendly summaries of accomplishments

### Performance Optimization
- **Command Classification**: Simple/Moderate/Complex categorization
- **Fast Path Execution**: <2 seconds for simple commands
- **Smart Caching**: Explanations and validations cached for repeated operations
- **Adaptive Timeouts**: Different timeouts based on command complexity

### Error Recovery
- **Intelligent Analysis**: LLM-powered error classification and recovery
- **Context-Aware Recovery**: Strategies based on error type (permission, dependency, network)
- **Learning System**: Improves recovery strategies over time
- **Safety Validation**: Validates recovery commands before execution

## Key Features

### Dual Mode Operation
- **Agent Mode**: Autonomous task execution with multi-agent workflow
- **Chat Mode**: Conversational DevOps guidance and assistance

### Real-Time Communication
- **WebSocket Streaming**: Real-time updates and progress tracking
- **Visual Feedback**: Agent status indicators and phase transitions
- **Terminal Integration**: Live command output streaming

### DevOps Specialization
- Infrastructure & Cloud (AWS, GCP, Azure, Terraform)
- Containerization (Docker, Kubernetes, Helm)
- CI/CD (GitHub Actions, GitLab CI, Jenkins)
- Monitoring (Prometheus, Grafana, ELK Stack)
- Security (SAST, DAST, vulnerability scanning)

## Development Guidelines

### Code Organization
- **Backend**: Modular agent system with clear separation of concerns
- **Frontend**: Component-based architecture with WebSocket communication
- **Testing**: Unit tests for agents, integration tests for workflows
- **Documentation**: Comprehensive flow documentation and API specs

### Performance Requirements
- Simple commands: <2 seconds execution time
- Moderate commands: <10 seconds with caching
- Complex commands: Optimized but full analysis retained
- Real-time streaming: Sub-second WebSocket message delivery

### Security Considerations
- Command validation and sanitization
- Secure handling of elevated privileges
- API key management and rotation
- Input validation for all user inputs

## File Structure Context

### Backend Structure
```
backend/
├── main.py                 # FastAPI application and WebSocket handlers
├── terminal_manager.py     # Terminal process management
├── devops_agent.py        # Chat mode DevOps agent
└── agent/                 # Multi-agent system
    ├── workflow_coordinator.py    # Orchestrates 4-phase workflow
    ├── understanding_agent.py     # Request analysis
    ├── planner.py                # Command planning
    ├── executor.py               # Command execution
    ├── error_recovery_agent.py   # Error analysis and recovery
    ├── summarizer_agent.py       # Workflow summarization
    └── schemas.py                # Data models and validation
```

### Frontend Structure
```
frontend/
├── src/
    ├── main.js                    # Electron main process
    ├── chat.js                    # Chat interface and agent communication
    ├── terminal.js                # Terminal emulation and management
    ├── agent_status_manager.js    # Visual feedback for agent phases
    └── index.html                 # Main application UI
```

## Common Patterns and Conventions

### WebSocket Message Protocol
```javascript
{
  type: "workflow_started" | "step_started" | "step_completed" | ...,
  data: { /* event-specific data */ },
  timestamp: "ISO-8601 timestamp"
}
```

### Agent Response Format
All agents return structured JSON responses with validation and fallback mechanisms.

### Error Handling
- Graceful degradation for LLM failures
- Comprehensive error recovery strategies
- User-friendly error messages and guidance

### Performance Optimization
- Command classification before execution
- Caching for repeated operations
- Fast paths for simple commands
- Adaptive monitoring based on complexity

This context should inform all development decisions and ensure consistency across the codebase.