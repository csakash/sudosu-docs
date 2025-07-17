---
inclusion: always
---

# Development Standards for AI Terminal Copilot

## Code Quality Standards

### Python Backend Standards

#### Code Style
- **PEP 8 Compliance**: Follow Python PEP 8 style guidelines
- **Type Hints**: Use type hints for all function parameters and return values
- **Docstrings**: Use Google-style docstrings for all classes and functions
- **Line Length**: Maximum 100 characters per line
- **Import Organization**: Group imports (standard library, third-party, local)

#### Example Function Structure
```python
async def execute_command(
    self, 
    command: str, 
    timeout: int = 60,
    callback: Optional[Callable] = None
) -> Dict[str, Any]:
    """
    Execute a terminal command with monitoring and callback support.
    
    Args:
        command: The terminal command to execute
        timeout: Maximum execution time in seconds
        callback: Optional callback for progress updates
        
    Returns:
        Dictionary containing execution results with output, exit_code, and error
        
    Raises:
        TerminalException: If terminal process is not available
        TimeoutError: If command execution exceeds timeout
    """
```

#### Error Handling
- **Specific Exceptions**: Use specific exception types, not generic Exception
- **Graceful Degradation**: Always provide fallback mechanisms
- **Logging**: Use structured logging with appropriate levels
- **User-Friendly Messages**: Convert technical errors to user-friendly messages

#### Async/Await Patterns
- **Consistent Usage**: Use async/await consistently throughout
- **Resource Cleanup**: Always use try/finally or context managers
- **Timeout Handling**: Implement proper timeout mechanisms
- **Cancellation Support**: Support task cancellation where appropriate

### JavaScript Frontend Standards

#### Code Style
- **ES6+ Features**: Use modern JavaScript features (async/await, destructuring, etc.)
- **Consistent Naming**: Use camelCase for variables and functions
- **Event Handling**: Use proper event listener management
- **Error Handling**: Implement comprehensive error handling

#### WebSocket Communication
```javascript
// Standard WebSocket message handling pattern
async function handleWebSocketMessage(event) {
    try {
        const data = JSON.parse(event.data);
        
        switch (data.type) {
            case 'workflow_started':
                await handleWorkflowStarted(data.data);
                break;
            case 'step_completed':
                await handleStepCompleted(data.data);
                break;
            default:
                console.warn('Unknown message type:', data.type);
        }
    } catch (error) {
        console.error('Error handling WebSocket message:', error);
        displayErrorMessage('Communication error occurred');
    }
}
```

#### UI Component Standards
- **Responsive Design**: Ensure components work across different screen sizes
- **Accessibility**: Include proper ARIA labels and keyboard navigation
- **Visual Feedback**: Provide clear feedback for all user actions
- **Loading States**: Show appropriate loading indicators

## Architecture Standards

### Multi-Agent System Design

#### Agent Interface Consistency
All agents should implement consistent interfaces:

```python
class BaseAgent:
    async def process(
        self, 
        input_data: Any, 
        callback: Optional[Callable] = None
    ) -> Dict[str, Any]:
        """Standard agent processing interface."""
        pass
    
    def _validate_input(self, input_data: Any) -> bool:
        """Validate input data structure."""
        pass
    
    def _create_fallback_response(self, error: str) -> Dict[str, Any]:
        """Create fallback response for failures."""
        pass
```

#### LLM Integration Standards
- **Prompt Engineering**: Use structured, consistent prompts
- **Response Validation**: Always validate LLM responses
- **Fallback Mechanisms**: Implement fallbacks for LLM failures
- **Streaming Support**: Support real-time streaming where appropriate

#### Performance Optimization
- **Command Classification**: Classify all commands by complexity
- **Caching Strategy**: Implement consistent caching patterns
- **Fast Path Logic**: Use fast paths for simple operations
- **Monitoring Integration**: Include performance monitoring

### WebSocket Communication Standards

#### Message Structure
```typescript
interface WebSocketMessage {
    type: string;           // Event type identifier
    data: any;             // Event-specific payload
    timestamp?: string;     // ISO-8601 timestamp
    workflow_id?: string;   // Optional workflow identifier
    step_id?: number;      // Optional step identifier
}
```

#### Event Types
- **Workflow Events**: `workflow_started`, `workflow_completed`, `workflow_failed`
- **Phase Events**: `understanding_started`, `planning_completed`, etc.
- **Step Events**: `step_started`, `step_completed`, `step_failed`
- **Recovery Events**: `recovery_started`, `recovery_completed`
- **Terminal Events**: `command_start`, `command_end`, `output`

#### Error Handling
- **Connection Management**: Handle disconnections gracefully
- **Retry Logic**: Implement exponential backoff for reconnections
- **State Synchronization**: Maintain state consistency across connections
- **Timeout Handling**: Set appropriate timeouts for different operations

## Testing Standards

### Unit Testing

#### Backend Testing
```python
import pytest
from unittest.mock import Mock, AsyncMock
from your_module import YourClass

@pytest.mark.asyncio
async def test_command_execution_success():
    """Test successful command execution."""
    # Arrange
    terminal_manager = Mock()
    terminal_manager.execute_command = AsyncMock(
        return_value={"output": "success", "exit_code": 0}
    )
    
    # Act
    result = await your_function(terminal_manager, "test command")
    
    # Assert
    assert result["success"] is True
    assert "success" in result["output"]
    terminal_manager.execute_command.assert_called_once()
```

#### Frontend Testing
- **DOM Testing**: Test UI interactions and state changes
- **WebSocket Testing**: Mock WebSocket connections for testing
- **Event Testing**: Test event handling and propagation
- **Integration Testing**: Test component interactions

### Integration Testing

#### Workflow Testing
- **End-to-End Flows**: Test complete agent workflows
- **Error Scenarios**: Test error handling and recovery
- **Performance Testing**: Validate response time requirements
- **Concurrent Testing**: Test multiple simultaneous operations

#### API Testing
- **WebSocket Testing**: Test real-time communication
- **Session Management**: Test session creation and cleanup
- **Error Handling**: Test API error responses
- **Authentication**: Test security mechanisms

## Performance Standards

### Response Time Requirements
- **Simple Commands**: <2 seconds total execution time
- **Moderate Commands**: <10 seconds with caching benefits
- **Complex Commands**: Optimized execution with progress feedback
- **WebSocket Messages**: <100ms delivery time
- **UI Updates**: <50ms response to user interactions

### Resource Management
- **Memory Usage**: Monitor and limit memory consumption
- **Connection Limits**: Manage WebSocket connection limits
- **Cache Management**: Implement proper cache eviction
- **Process Cleanup**: Ensure proper cleanup of terminal processes

### Monitoring and Metrics
- **Execution Times**: Track command execution performance
- **Cache Hit Rates**: Monitor caching effectiveness
- **Error Rates**: Track and analyze error patterns
- **User Experience**: Monitor user interaction patterns

## Security Standards

### Input Validation
- **Command Sanitization**: Validate and sanitize all terminal commands
- **Parameter Validation**: Validate all API parameters
- **XSS Prevention**: Sanitize all user-generated content
- **Injection Prevention**: Prevent command injection attacks

### Authentication and Authorization
- **API Key Management**: Secure storage and rotation of API keys
- **Session Security**: Implement secure session management
- **Privilege Escalation**: Handle sudo operations securely
- **Resource Access**: Control access to system resources

### Data Protection
- **Sensitive Data**: Avoid logging sensitive information
- **Error Messages**: Limit sensitive information in error messages
- **Communication Security**: Use secure WebSocket connections
- **Storage Security**: Secure storage of session data

## Documentation Standards

### Code Documentation
- **Inline Comments**: Explain complex logic and business rules
- **API Documentation**: Document all public APIs and interfaces
- **Architecture Documentation**: Maintain up-to-date architecture docs
- **Flow Documentation**: Document all major system flows

### User Documentation
- **Setup Instructions**: Clear installation and setup guides
- **Usage Examples**: Provide practical usage examples
- **Troubleshooting**: Common issues and solutions
- **Feature Documentation**: Comprehensive feature explanations

### Development Documentation
- **Contributing Guidelines**: How to contribute to the project
- **Development Setup**: Local development environment setup
- **Testing Guidelines**: How to run and write tests
- **Deployment Instructions**: How to deploy the application

These standards ensure consistency, maintainability, and quality across the entire AI Terminal Copilot codebase.