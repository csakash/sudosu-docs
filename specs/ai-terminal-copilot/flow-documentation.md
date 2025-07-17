# AI Terminal Copilot - Flow Documentation

## Overview

This document provides detailed flow diagrams and explanations for the three core interaction patterns in the AI Terminal Copilot system. Each flow demonstrates how user requests are processed through different pathways depending on the mode and complexity of the operation.

---

## 1. Agent Mode Flow

### Flow Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend (Electron)
    participant WS as WebSocket Handler
    participant WC as Workflow Coordinator
    participant UA as Understanding Agent
    participant PA as Planning Agent
    participant EA as Execution Agent
    participant SA as Summarization Agent
    participant TM as Terminal Manager
    participant PTY as System Terminal

    U->>FE: Types "Create nginx proxy from 8000 to 80" + Click Run
    FE->>FE: Check currentMode === 'agent'
    FE->>FE: Display user message in chat
    FE->>FE: Start AgentStatusManager (workflow phase)
    FE->>WS: WebSocket message {type: "start_workflow", objective, terminal_session_id}
    
    WS->>WC: execute_complete_workflow(objective, terminal_session_id, callback)
    WC->>FE: callback({type: "workflow_started"})
    FE->>FE: Update status to "🤖 Starting enhanced agent workflow..."
    
    Note over WC,SA: PHASE 1: UNDERSTANDING
    WC->>UA: analyze_user_request(objective, callback)
    UA->>UA: Send to Google Gemini with understanding prompt
    loop Streaming Response
        UA->>FE: callback({type: "understanding_progress", chunk})
        FE->>FE: Update status to "🧠 Analyzing your request..."
    end
    UA->>UA: Parse JSON response (intent, complexity, technologies, challenges)
    UA->>WC: Return understanding_result
    WC->>FE: callback({type: "understanding_phase_completed", analysis})
    
    Note over WC,SA: PHASE 2: PLANNING
    WC->>PA: create_plan(objective, terminal_session_id, understanding_result, callback)
    PA->>PA: Enhanced prompt with understanding context
    PA->>PA: Send to Google Gemini for step generation
    loop Streaming Response
        PA->>FE: callback({type: "planning_progress", chunk})
        FE->>FE: Update status to "📋 Creating execution plan..."
    end
    PA->>PA: Parse JSON response into Plan object with Steps
    PA->>WC: Return plan
    WC->>FE: callback({type: "planning_phase_completed", plan})
    FE->>FE: Display execution plan with step indicators
    
    Note over WC,SA: PHASE 3: EXECUTION
    WC->>EA: _execute_execution_phase(plan, terminal_session_id, callback)
    EA->>FE: callback({type: "execution_phase_started"})
    FE->>FE: Update status to "⚡ Executing commands..."
    
    loop For Each Step in Plan
        EA->>EA: Classify command complexity (Simple/Moderate/Complex)
        EA->>FE: callback({type: "step_started", step_id, description, command})
        FE->>FE: Update step indicator to "running"
        
        EA->>TM: monitor_command(step.command, terminal, timeout)
        TM->>PTY: write(command + '\n')
        
        loop Command Execution
            PTY->>TM: read() - command output
            TM->>FE: WebSocket structured message {type: "output", data: output}
            FE->>FE: Display output in terminal interface
        end
        
        TM->>EA: Return CommandResult {success, output, exit_code}
        
        alt Command Success
            EA->>EA: Validate command result
            EA->>FE: callback({type: "step_completed", status: "completed"})
            FE->>FE: Update step indicator to "✅ completed"
        else Command Failure
            EA->>EA: Attempt intelligent recovery
            EA->>FE: callback({type: "intelligent_recovery_started"})
            EA->>EA: Generate recovery strategy
            EA->>TM: Execute recovery commands
            alt Recovery Success
                EA->>FE: callback({type: "step_completed", status: "completed"})
                FE->>FE: Update step indicator to "✅ completed"
            else Recovery Failure
                EA->>FE: callback({type: "step_completed", status: "failed"})
                FE->>FE: Update step indicator to "❌ failed"
            end
        end
    end
    
    EA->>WC: Return execution_results
    WC->>FE: callback({type: "execution_phase_completed"})
    
    Note over WC,SA: PHASE 4: SUMMARIZATION
    WC->>SA: summarize_workflow(request, understanding, plan, results, callback)
    SA->>SA: Build context from all phases
    SA->>SA: Send to Google Gemini with summary prompt
    loop Streaming Response
        SA->>FE: callback({type: "summarizing_progress", chunk})
        FE->>FE: Update status to "📝 Generating summary..."
    end
    SA->>SA: Parse JSON response (friendly summary, accomplishments, next steps)
    SA->>WC: Return final_summary
    WC->>FE: callback({type: "workflow_completed", summary})
    FE->>FE: Display friendly summary in chat
    FE->>FE: AgentStatusManager.stopAgentActivity() - "🎉 Agent execution completed!"
```

### Detailed Step-by-Step Process

#### Frontend Initiation
1. **User Input**: User types request in agent mode and clicks "Run"
2. **Mode Check**: Frontend verifies `currentMode === 'agent'`
3. **UI Updates**: Display user message, start agent status manager
4. **WebSocket Send**: Send workflow start message to backend

#### Backend Workflow Coordination
5. **Workflow Creation**: Generate unique workflow ID and initialize tracking
6. **Phase Orchestration**: Execute 4-phase workflow with streaming callbacks

#### Phase 1: Understanding (2-5 seconds)
7. **LLM Analysis**: Send user request to Google Gemini with specialized understanding prompt
8. **Streaming Response**: Real-time token streaming to frontend
9. **JSON Parsing**: Parse structured response with intent, complexity, technologies
10. **Context Building**: Create enhanced context for planning phase

#### Phase 2: Planning (3-8 seconds)
11. **Enhanced Prompting**: Combine user request with understanding context
12. **Command Generation**: LLM generates step-by-step terminal commands
13. **Plan Validation**: Validate command structure and safety
14. **Plan Object Creation**: Convert to structured Plan with Step objects

#### Phase 3: Execution (Variable duration)
15. **Step Iteration**: Execute each step with monitoring and feedback
16. **Command Classification**: Determine complexity for performance optimization
17. **Terminal Execution**: Send commands to actual terminal process
18. **Real-time Output**: Stream terminal output to frontend
19. **Error Recovery**: Intelligent failure analysis and recovery attempts
20. **Progress Tracking**: Update step indicators and status

#### Phase 4: Summarization (2-4 seconds)
21. **Context Aggregation**: Combine results from all phases
22. **Summary Generation**: Create human-friendly explanation
23. **Final Display**: Show accomplishments, next steps, and important details

---

## 2. Chat Mode Flow

### Flow Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend (Electron)
    participant WS as Chat WebSocket Handler
    participant DA as DevOps Agent
    participant LLM as Google Gemini
    participant CS as Chat Session Store

    U->>FE: Types "How do I configure SSL for nginx?" + Click Run
    FE->>FE: Check currentMode === 'chat'
    FE->>FE: Display user message in chat
    
    FE->>WS: WebSocket message {type: "user_message", content: message}
    WS->>CS: Add user message to session history
    WS->>FE: Send acknowledgment {type: "message_received"}
    
    WS->>DA: get_response(message, chat_history, websocket)
    DA->>DA: Build conversation context with system prompt
    DA->>DA: Add last 10 messages from history for context
    DA->>DA: Append current user message
    
    DA->>LLM: Send messages with DevOps expertise prompt
    Note over DA,LLM: Streaming Response Generation
    
    loop Streaming Tokens
        LLM->>DA: Stream response tokens
        DA->>WS: StreamingCallbackHandler.on_llm_new_token()
        WS->>FE: WebSocket {type: "token", content: token}
        FE->>FE: Append token to current message display
    end
    
    LLM->>DA: Complete response
    DA->>WS: StreamingCallbackHandler.on_llm_end()
    WS->>FE: WebSocket {type: "end"}
    FE->>FE: Finalize message display
    
    DA->>WS: Return complete response text
    WS->>CS: Add AI response to session history
    WS->>FE: Final response confirmation
    
    Note over FE: User sees complete DevOps guidance with:
    Note over FE: - Code blocks with Copy/Apply buttons
    Note over FE: - Step-by-step instructions
    Note over FE: - Best practices and security considerations
```

### Detailed Step-by-Step Process

#### Frontend Chat Interaction
1. **User Input**: User types DevOps question in chat mode
2. **Mode Verification**: Frontend confirms `currentMode === 'chat'`
3. **Message Display**: Show user message in chat interface
4. **WebSocket Communication**: Send message to chat WebSocket handler

#### Backend Chat Processing
5. **Session Management**: Retrieve or create chat session
6. **History Management**: Add user message to persistent session history
7. **Context Building**: Prepare conversation context with system prompt
8. **DevOps Agent Invocation**: Call specialized DevOps agent

#### DevOps Agent Response Generation
9. **Conversation Context**: Build messages array with system prompt and history
10. **LLM Integration**: Send to Google Gemini with DevOps expertise prompt
11. **Streaming Response**: Real-time token streaming via callback handler
12. **Domain Expertise**: Provide specialized knowledge in:
    - Infrastructure & Cloud (AWS, GCP, Azure, Terraform)
    - Containerization (Docker, Kubernetes, Helm)
    - CI/CD (GitHub Actions, GitLab CI, Jenkins)
    - Monitoring (Prometheus, Grafana, ELK Stack)
    - Security (SAST, DAST, vulnerability scanning)

#### Frontend Response Display
13. **Real-time Rendering**: Display streaming tokens as they arrive
14. **Code Block Processing**: Render code blocks with Copy/Apply buttons
15. **Markdown Formatting**: Apply proper formatting for readability
16. **Session Persistence**: Maintain conversation history for context

### Chat Mode Features

#### Code Block Functionality
```javascript
// Copy button functionality
window.copyToClipboard = function(button) {
    const codeBlock = button.parentElement.querySelector('code');
    navigator.clipboard.writeText(codeBlock.textContent);
    // Visual feedback: "Copied!" with green background
};

// Apply button functionality  
window.applyToTerminal = function(button) {
    const codeBlock = button.parentElement.querySelector('code');
    const socket = getSocket();
    socket.send(codeBlock.textContent.trimEnd() + '\n');
    // Visual feedback: "Applied!" with green background
};
```

#### Session Management
- **Multiple Sessions**: Users can create and switch between chat sessions
- **Persistent History**: Conversations saved and restored across sessions
- **Context Preservation**: Maintain conversation context for better responses
- **Session Cleanup**: Proper resource management and cleanup

---

## 3. Complete Multi-Agent Flow

### Comprehensive System Architecture Flow

```mermaid
graph TB
    subgraph "Frontend Layer"
        UI[User Interface]
        Chat[Chat Panel]
        Terminal[Terminal Panel]
        Status[Agent Status Manager]
        
        UI --> Chat
        UI --> Terminal
        UI --> Status
    end
    
    subgraph "Communication Layer"
        ChatWS[Chat WebSocket]
        TerminalWS[Terminal WebSocket]
        AgentWS[Agent WebSocket]
        
        Chat --> ChatWS
        Terminal --> TerminalWS
        Status --> AgentWS
    end
    
    subgraph "Backend API Layer"
        FastAPI[FastAPI Server]
        WSHandlers[WebSocket Handlers]
        RESTEndpoints[REST Endpoints]
        
        ChatWS --> WSHandlers
        TerminalWS --> WSHandlers
        AgentWS --> WSHandlers
        WSHandlers --> FastAPI
        RESTEndpoints --> FastAPI
    end
    
    subgraph "Workflow Orchestration"
        WC[Workflow Coordinator]
        SessionMgr[Session Manager]
        
        FastAPI --> WC
        FastAPI --> SessionMgr
    end
    
    subgraph "Multi-Agent System"
        UA[Understanding Agent]
        PA[Planning Agent]
        EA[Execution Agent]
        SA[Summarization Agent]
        RA[Recovery Agent]
        DA[DevOps Agent]
        
        WC --> UA
        UA --> PA
        PA --> EA
        EA --> SA
        EA --> RA
        WSHandlers --> DA
    end
    
    subgraph "Performance & Optimization"
        PO[Performance Optimizer]
        Cache[Caching System]
        CM[Command Monitor]
        
        EA --> PO
        EA --> Cache
        EA --> CM
    end
    
    subgraph "Terminal Management"
        TM[Terminal Manager]
        PTY[PTY Process]
        
        EA --> TM
        TM --> PTY
        TerminalWS --> TM
    end
    
    subgraph "External Services"
        Gemini[Google Gemini LLM]
        
        UA --> Gemini
        PA --> Gemini
        SA --> Gemini
        RA --> Gemini
        DA --> Gemini
    end
    
    subgraph "Data Flow"
        Request[User Request]
        Understanding[Understanding Result]
        Plan[Execution Plan]
        Results[Execution Results]
        Summary[Final Summary]
        
        Request --> Understanding
        Understanding --> Plan
        Plan --> Results
        Results --> Summary
    end
```

### Multi-Agent Interaction Flow

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant WorkflowCoordinator
    participant UnderstandingAgent
    participant PlanningAgent
    participant ExecutionAgent
    participant RecoveryAgent
    participant SummarizationAgent
    participant TerminalManager
    participant GoogleGemini

    User->>Frontend: Submit request in Agent Mode
    Frontend->>WorkflowCoordinator: Start workflow with objective
    
    Note over WorkflowCoordinator: Initialize workflow tracking
    WorkflowCoordinator->>Frontend: Workflow started event
    
    rect rgb(200, 230, 255)
        Note over UnderstandingAgent: PHASE 1: UNDERSTANDING
        WorkflowCoordinator->>UnderstandingAgent: Analyze user request
        UnderstandingAgent->>GoogleGemini: Send understanding prompt
        GoogleGemini-->>UnderstandingAgent: Stream analysis response
        UnderstandingAgent->>Frontend: Stream understanding progress
        UnderstandingAgent->>WorkflowCoordinator: Return structured analysis
        WorkflowCoordinator->>Frontend: Understanding phase completed
    end
    
    rect rgb(200, 255, 200)
        Note over PlanningAgent: PHASE 2: PLANNING
        WorkflowCoordinator->>PlanningAgent: Create execution plan
        PlanningAgent->>GoogleGemini: Send enhanced planning prompt
        GoogleGemini-->>PlanningAgent: Stream plan response
        PlanningAgent->>Frontend: Stream planning progress
        PlanningAgent->>WorkflowCoordinator: Return structured plan
        WorkflowCoordinator->>Frontend: Planning phase completed
    end
    
    rect rgb(255, 230, 200)
        Note over ExecutionAgent: PHASE 3: EXECUTION
        WorkflowCoordinator->>ExecutionAgent: Execute plan steps
        ExecutionAgent->>Frontend: Execution phase started
        
        loop For each step in plan
            ExecutionAgent->>ExecutionAgent: Classify command complexity
            ExecutionAgent->>Frontend: Step started event
            ExecutionAgent->>TerminalManager: Execute command
            TerminalManager->>Frontend: Stream command output
            
            alt Command succeeds
                ExecutionAgent->>Frontend: Step completed successfully
            else Command fails
                ExecutionAgent->>RecoveryAgent: Analyze error
                RecoveryAgent->>GoogleGemini: Generate recovery strategy
                GoogleGemini-->>RecoveryAgent: Return recovery steps
                RecoveryAgent->>ExecutionAgent: Provide recovery plan
                ExecutionAgent->>TerminalManager: Execute recovery commands
                
                alt Recovery succeeds
                    ExecutionAgent->>TerminalManager: Re-execute original command
                    ExecutionAgent->>Frontend: Step completed after recovery
                else Recovery fails
                    ExecutionAgent->>Frontend: Step failed after recovery attempts
                end
            end
        end
        
        ExecutionAgent->>WorkflowCoordinator: Return execution results
        WorkflowCoordinator->>Frontend: Execution phase completed
    end
    
    rect rgb(255, 200, 255)
        Note over SummarizationAgent: PHASE 4: SUMMARIZATION
        WorkflowCoordinator->>SummarizationAgent: Summarize workflow
        SummarizationAgent->>GoogleGemini: Generate friendly summary
        GoogleGemini-->>SummarizationAgent: Stream summary response
        SummarizationAgent->>Frontend: Stream summarization progress
        SummarizationAgent->>WorkflowCoordinator: Return final summary
        WorkflowCoordinator->>Frontend: Workflow completed with summary
    end
    
    Frontend->>User: Display complete workflow results
```

### Agent Specialization and Responsibilities

#### Understanding Agent
- **Primary Function**: Request analysis and intent recognition
- **LLM Prompt**: Specialized for breaking down complex DevOps requests
- **Output**: Structured JSON with objectives, complexity, and context
- **Performance**: 2-5 seconds for analysis
- **Key Features**:
  - Technology identification
  - Complexity assessment
  - Prerequisite analysis
  - Challenge identification

#### Planning Agent
- **Primary Function**: Convert objectives to executable commands
- **LLM Prompt**: Specialized for terminal command generation
- **Output**: Structured plan with step-by-step commands
- **Performance**: 3-8 seconds for planning
- **Key Features**:
  - Command sequence generation
  - Step descriptions
  - Safety validation
  - Context integration

#### Execution Agent
- **Primary Function**: Command execution with monitoring
- **Technology**: Async Python with performance optimization
- **Performance**: Variable based on command complexity
- **Key Features**:
  - Command classification (Simple/Moderate/Complex)
  - Fast path execution (<2 seconds for simple commands)
  - Real-time monitoring and feedback
  - Intelligent retry mechanisms
  - Error recovery integration

#### Recovery Agent
- **Primary Function**: Error analysis and recovery strategy generation
- **LLM Prompt**: Specialized for troubleshooting and recovery
- **Output**: Recovery steps with safety validation
- **Key Features**:
  - Error classification (permission, dependency, network, resource)
  - Context-aware recovery strategies
  - Safety validation for recovery commands
  - Learning from successful recoveries

#### Summarization Agent
- **Primary Function**: Human-friendly workflow explanation
- **LLM Prompt**: Conversational and friendly tone
- **Output**: Accomplishments, next steps, important details
- **Performance**: 2-4 seconds for summary generation
- **Key Features**:
  - Natural language explanations
  - Context aggregation from all phases
  - Actionable next steps
  - Important detail highlighting

### Performance Optimization Flow

```mermaid
graph TD
    A[Command Received] --> B{Classify Complexity}
    B -->|Simple| C[Fast Path]
    B -->|Moderate| D[Cached Path]
    B -->|Complex| E[Full Analysis]
    
    C --> F[Heuristic Validation]
    C --> G[<2 Second Execution]
    
    D --> H[Check Cache]
    H -->|Hit| I[Use Cached Result]
    H -->|Miss| J[Generate & Cache]
    D --> K[<10 Second Execution]
    
    E --> L[Full LLM Analysis]
    E --> M[Complete Monitoring]
    E --> N[Optimized Execution]
    
    F --> O[Success]
    I --> O
    J --> O
    L --> O
    
    G --> O
    K --> O
    N --> O
```

### Error Recovery Flow

```mermaid
graph TD
    A[Command Fails] --> B[Error Analysis]
    B --> C{Error Classification}
    
    C -->|Permission| D[Permission Recovery]
    C -->|Dependency| E[Dependency Recovery]
    C -->|Network| F[Network Recovery]
    C -->|Resource| G[Resource Recovery]
    C -->|Unknown| H[Generic Recovery]
    
    D --> I[sudo escalation]
    D --> J[File permissions]
    
    E --> K[Package installation]
    E --> L[Command availability]
    
    F --> M[Connectivity test]
    F --> N[DNS resolution]
    
    G --> O[Disk space check]
    G --> P[Memory check]
    
    H --> Q[Generic troubleshooting]
    
    I --> R[Re-execute Original]
    J --> R
    K --> R
    L --> R
    M --> R
    N --> R
    O --> R
    P --> R
    Q --> R
    
    R --> S{Success?}
    S -->|Yes| T[Continue Workflow]
    S -->|No| U[Manual Resolution]
```

This comprehensive flow documentation provides a complete understanding of how the AI Terminal Copilot processes different types of user interactions through its sophisticated multi-agent architecture, ensuring optimal performance and user experience across all scenarios.