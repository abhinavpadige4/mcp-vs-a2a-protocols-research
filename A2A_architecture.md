# Agent2Agent (A2A) Protocol - Architecture Deep Dive

## Overview
The Agent2Agent (A2A) Protocol is an open standard that enables inter-agent communication and collaboration, allowing AI agents from different frameworks or vendors to discover each other, exchange capabilities, delegate tasks, and work together on complex workflows. Launched by Google in April 2025, A2A focuses on the horizontal layer of agent-to-agent interaction.

## Core Architecture Components

### 1. Client Agent (Task Initiator)
The Client Agent is responsible for initiating tasks and managing the workflow delegation process. It acts as the orchestrator or requester in agent-to-agent interactions.

**Key Responsibilities:**
- Discover available Remote Agents in the ecosystem
- Negotiate capabilities and establish communication channels
- Delegate tasks to appropriate Remote Agents
- Monitor task progress and handle intermediate results
- Aggregate final results from delegated work
- Handle error recovery and retry mechanisms

### 2. Remote Agent (Task Executor)
The Remote Agent executes tasks delegated by Client Agents and returns results. It represents a capable AI agent that can perform specific functions or services.

**Key Responsibilities:**
- Advertise capabilities and services to potential clients
- Accept and validate incoming task requests
- Execute assigned tasks using internal AI capabilities
- Provide status updates and progress reporting
- Return results in standardized format
- Handle resource management and cleanup

## Communication Model

### Transport Layer
Unlike MCP, A2A mandates **HTTP with JSON payloads** as its primary transport mechanism:
- **HTTP/POST**: For synchronous request-response interactions
- **HTTP/GET**: For capability discovery and status polling
- **HTTP Streaming**: For long-running tasks with progressive results
- **WebSocket**: Optional for real-time bidirectional communication
- **RESTful Principles**: Standard HTTP methods and status codes

### Message Format
A2A uses **structured JSON over HTTP** with well-defined schemas for all interactions:
- **Content-Type**: `application/json`
- **Character Encoding**: UTF-8
- **Schema Validation**: Strict JSON Schema validation for all messages
- **Versioning**: Explicit API versioning in URL paths or headers

## Core Interactions

### 1. Agent Discovery
Client Agents discover Remote Agents through various mechanisms:

#### Well-Known Endpoint
```
GET /.well-known/agent-card
```
Returns agent metadata including:
- Agent ID, name, description
- Supported capabilities and skills
- Communication endpoints
- Authentication requirements
- Version information

#### Discovery Services
- Centralized agent registries
- Peer-to-peer discovery protocols
- DNS-based service discovery
- Cloud provider service meshes

### 2. Capability Negotiation
Before task delegation, agents negotiate compatible capabilities:

#### Capability Exchange
Clients send `GET /capabilities` to discover:
- Supported task types and categories
- Input/output schema definitions
- Performance characteristics
- Security requirements
- Resource requirements

#### Schema Negotiation
Agents exchange JSON Schemas to validate:
- Task request formats
- Result formats
- Intermediate update formats
- Error report formats

### 3. Task Delegation
The core A2A interaction pattern for work distribution:

#### Task Submission
```
POST /tasks
Content-Type: application/json

{
  "taskId": "unique-task-identifier",
  "skill": "specific-capability-name",
  "input": {...},
  "metadata": {
    "priority": "high",
    "timeout": 300,
    "retryCount": 3
  }
}
```

#### Task Acknowledgment
Immediate response confirming task acceptance:
```json
{
  "taskId": "unique-task-identifier",
  "status": "accepted",
  "estimatedCompletion": "timestamp"
}
```

#### Status Streaming
For long-running tasks, agents can use:
- **Polling**: `GET /tasks/{taskId}/status`
- **Streaming**: `GET /tasks/{taskId}/events` (Server-Sent Events)
- **WebSocket**: Real-time status updates

#### Result Retrieval
Upon completion:
```
GET /tasks/{taskId}/result
```
Returns:
```json
{
  "taskId": "unique-task-identifier",
  "status": "completed|failed",
  "result": {...},
  "artifacts": [...],
  "metrics": {
    "executionTime": 1250,
    "tokensUsed": 8450
  }
}
```

### 4. Result Aggregation
Client Agents manage complex workflows by:

#### Parallel Execution
- Delegating multiple tasks simultaneously
- Tracking completion of all subtasks
- Handling partial failures gracefully

#### Sequential Workflows
- Chaining tasks where output of one feeds into next
- Managing dependencies between agent interactions
- Handling intermediate data transformation

#### Fan-Out/Fan-In Patterns
- Distributing work to multiple agents
- Collecting and synthesizing results
- Applying reduction or voting mechanisms

## Security Model

### Transport Security
- **HTTPS Mandatory**: All A2A communication must use TLS 1.2+
- **Certificate Validation**: Strict hostname and certificate chain validation
- **Perfect Forward Secrecy**: Ephemeral key exchanges for forward security

### Authentication and Authorization
- **API Key Authentication**: Simple key-based auth for basic scenarios
- **OAuth 2.0**: Industry-standard token-based authentication
- **Mutual TLS**: Certificate-based authentication for high-security scenarios
- **JWT Tokens**: Stateless authentication with claims and expiration
- **Custom Auth Schemes**: Extensible framework for proprietary mechanisms

### Authorization Model
- **Role-Based Access Control (RBAC)**: Predefined roles with permissions
- **Attribute-Based Access Control (ABAC)**: Dynamic policies based on attributes
- **Scope-Based Permissions**: Fine-grained control over specific capabilities
- **Delegation Tokens**: Limited-use tokens for specific task delegations

### Data Protection
- **Message Encryption**: TLS encryption for data in transit
- **Input Validation**: Strict JSON Schema validation prevents injection
- **Output Sanitization**: Server-side validation of responses
- **Audit Logging**: Comprehensive logging with tamper-evident storage
- **Data Minimization**: Principle of collecting only necessary data

### Rate Limiting and Abuse Prevention
- **Per-Client Limits**: Request quotas based on client identity
- **Per-Skill Limits**: Quotas for specific capabilities
- **Global Rate Limits**: System-wide protection against overload
- **Burst Allowances**: Short-term excess capacity for spikes
- **Graduated Responses**: Warnings → throttling → blocking escalation

## Interaction Patterns

### Simple Task Delegation
1. Client discovers Remote Agent via `/.well-known/agent-card`
2. Client validates agent capabilities match requirements
3. Client submits task via `POST /tasks`
4. Remote Agent acknowledges and begins execution
5. Client monitors progress via status endpoints
6. Remote Agent completes task and returns result
7. Client processes result and continues workflow

### Complex Workflow Orchestration
1. Client analyzes complex goal and decomposes into subtasks
2. Client discovers specialized agents for each subtask type
3. Client delegates tasks in parallel or sequence as needed
4. Client manages dependencies and data flow between tasks
5. Client handles failures with retry, fallback, or escalation
6. Client aggregates results and synthesizes final output
7. Client reports overall workflow completion status

### Peer-to-Peer Collaboration
1. Agents establish mutual discovery and trust relationships
2. Agents negotiate bidirectional capability exchange
3. Agents delegate work back and forth as needed
4. Agents maintain shared context and state synchronization
5. Agents resolve conflicts through predefined protocols
6. Agents dynamically adjust collaboration patterns

## Implementation Details

### Remote Agent Development
Remote Agents typically implement:
- HTTP server with RESTful endpoints
- JSON Schema validation for all requests/responses
- Task queuing and execution engine
- Progress tracking and status reporting
- Result formatting and artifact generation
- Security middleware and authentication
- Health checks and monitoring endpoints
- Graceful shutdown and cleanup procedures

### Client Agent Development
Client Agents typically implement:
- HTTP client with connection pooling
- Discovery mechanisms and agent caching
- Task delegation and workflow orchestration
- Status monitoring and progress tracking
- Result aggregation and synthesis
- Error handling and retry logic
- Security credential management
- Workflow persistence and recovery

### SDK Support
Official SDKs are available for:
- **Java**: Enterprise reference implementation
- **Python**: Rapid development and prototyping
- **Go**: High-performance microservices
- **Node.js**: JavaScript/TypeScript applications
- **Rust**: Systems-level agent implementations
- **.NET**: Microsoft ecosystem integration

## Error Handling
A2A defines standardized HTTP status codes and error formats:

### HTTP Status Codes
- `200 OK`: Successful request
- `202 Accepted`: Task accepted for processing
- `400 Bad Request`: Invalid request format or parameters
- `401 Unauthorized`: Authentication required or failed
- `403 Forbidden`: Authenticated but insufficient permissions
- `404 Not Found`: Agent, task, or resource not found
- `408 Request Timeout`: Request timed out
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server-side processing error
- `503 Service Unavailable`: Server temporarily unavailable

### Error Response Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR|AUTH_ERROR|TASK_ERROR|INTERNAL_ERROR",
    "message": "Human-readable error description",
    "details": {...},
    "timestamp": "ISO-8601 timestamp",
    "requestId": "correlation-id"
  }
}
```

## Extensibility
- **Custom Endpoints**: Agents can define proprietary APIs beyond core spec
- **Middleware Plugins**: Interceptors for logging, auth, transformation, metrics
- **Schema Extensions**: Extension mechanisms for custom data types
- **Protocol Versions**: Explicit versioning allows backward compatibility
- **Extension Registry**: Standardized way to publish and discover extensions

## Limitations and Considerations
- **Horizontal Focus**: Primarily addresses agent-to-agent collaboration
- **HTTP Overhead**: HTTP protocol overhead compared to binary protocols
- **Latency**: Additional network hops for multi-agent workflows
- **Complexity**: Managing distributed state and failure scenarios
- **Standardization**: Still evolving with active development
- **Ecosystem Maturity**: Newer protocol with growing adoption

## References
- Agent2Agent Protocol Specification v1.0
- Google Cloud A2A Documentation
- A2A Java SDK Source Code
- HTTP/1.1 Specification (RFC 7230-7235)
- JSON Schema Specification (https://json-schema.org/)