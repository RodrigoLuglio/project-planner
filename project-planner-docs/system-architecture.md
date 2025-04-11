# ProjectPlanner - System Architecture

This document provides a comprehensive overview of the ProjectPlanner system architecture, detailing component relationships, data flows, and technical considerations for implementation.

## System Overview

ProjectPlanner is a CLI application that orchestrates specialized AI agents to transform abstract ideas into comprehensive, implementation-ready project documentation. The system leverages Pydantic AI for agent management, type validation, and workflow coordination, with ADK-inspired patterns for enhanced agent collaboration.

### Key Features

- AI-driven project documentation generation
- Team of specialized agents with distinct responsibilities
- Self-contained stories ready for implementation
- Terminal-based user interface with rich formatting
- Extensible plugin architecture
- Themable interface with accessibility options
- Support for multiple LLM providers

## Architecture Principles

The architecture follows these core principles:

1. **Separation of Concerns**: Each component has a single, well-defined responsibility
2. **Type Safety**: Pydantic models ensure consistent data structures
3. **Extensibility**: Plugin architecture allows for easy extension
4. **Modular Design**: Components are loosely coupled through clean interfaces
5. **Progressive Enhancement**: Core functionality works with minimal dependencies
6. **Consistent Patterns**: Similar problems solved in similar ways

## High-Level Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  CLI Interface  │<--->│  Core Engine    │<--->│  Agent Manager  │
│                 │     │                 │     │   (Pydantic AI) │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                               ↑ ↓                     ↑ ↓
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Document Store │<--->│ Workflow Engine │<--->│   Agent Pool    │
│                 │     │ (Pydantic Graph)│     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                               ↑ ↓                     ↑ ↓
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│ Template Engine │<--->│  File Manager   │<--->│  Plugin System  │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Core Components

### 1. CLI Interface

The user-facing component that handles command processing and user interaction:

- Processes command-line arguments using Typer
- Formats output using Rich for terminal display
- Provides interactive prompts and dialogues
- Manages theme application and styling
- Handles error display and user feedback

#### Interactions
- **Provides to**: User commands to Core Engine
- **Receives from**: Formatted output from Core Engine

### 2. Core Engine

Central coordination component that manages application flow:

- Initializes and configures system components
- Dispatches commands to appropriate handlers
- Manages application state
- Coordinates component interactions
- Handles error propagation and recovery

#### Interactions
- **Provides to**: Coordination for all other components
- **Receives from**: Commands from CLI Interface, results from all components

### 3. Agent Manager

Manages AI agent lifecycle and interactions:

- Creates and configures agent instances
- Handles agent dependencies and tools
- Manages agent state and context
- Provides model provider abstraction
- Tracks token usage and quotas

#### Interactions
- **Provides to**: Agent instances to Workflow Engine and Core Engine
- **Receives from**: Agent configurations from Core Engine, execution requests from Workflow Engine

### 4. Document Store

Manages document storage, retrieval, and relationships:

- Stores and versions documents
- Validates document structures
- Manages document relationships and dependencies
- Provides search and indexing
- Handles document serialization and deserialization

#### Interactions
- **Provides to**: Documents to Core Engine, Template Engine, and Workflow Engine
- **Receives from**: Document data from agents, file content from File Manager

### 5. Workflow Engine

Orchestrates agent execution in defined workflows:

- Manages workflow definitions and transitions
- Handles workflow state
- Implements conditional branching
- Provides error handling and retry logic
- Manages workflow persistence

#### Interactions
- **Provides to**: Execution flow to Agent Manager, results to Core Engine
- **Receives from**: Workflow definitions from Core Engine, agent results from Agent Manager

### 6. Template Engine

Manages document templates and rendering:

- Loads and validates templates
- Renders templates with data
- Handles template variables and inheritance
- Provides template discovery
- Manages template versions

#### Interactions
- **Provides to**: Rendered documents to Document Store and File Manager
- **Receives from**: Template data from agents, template files from File Manager

### 7. File Manager

Handles file system operations:

- Creates and manages directory structures
- Reads and writes files
- Manages file permissions
- Provides path resolution
- Handles file metadata

#### Interactions
- **Provides to**: File content to Document Store and Template Engine
- **Receives from**: File paths from Core Engine, file content from Template Engine

### 8. Plugin System

Provides extensibility through plugins:

- Discovers and loads plugins
- Manages plugin lifecycle
- Provides plugin registration
- Handles plugin dependencies
- Validates plugin compatibility

#### Interactions
- **Provides to**: Plugin features to Core Engine and other components
- **Receives from**: Plugin configurations from Core Engine

### 9. Agent Pool

Manages specialized agents and their interactions:

- Provides specialized agent implementations
- Manages agent teams
- Coordinates agent delegation
- Handles agent specialization
- Provides agent discovery

#### Interactions
- **Provides to**: Specialized agents to Agent Manager
- **Receives from**: Agent requirements from Workflow Engine

## Data Models

### Core Data Models

#### 1. Project Configuration
```python
class ProjectConfig(BaseModel):
    name: str
    description: str
    created_at: datetime
    updated_at: datetime
    version: str
    workflows: List[str]
    default_workflow: str
    agents: Dict[str, AgentConfig]
```

#### 2. Document Model
```python
class DocumentMetadata(BaseModel):
    id: str
    title: str
    version: str
    created_at: datetime
    updated_at: datetime
    created_by: str
    tags: List[str] = []
    related_docs: List[str] = []

class Document(BaseModel):
    metadata: DocumentMetadata
    content: str
    sections: Dict[str, Any] = {}
```

#### 3. Workflow State
```python
class WorkflowState(BaseModel):
    workflow_id: str
    current_node: str
    started_at: datetime
    updated_at: datetime
    completed: bool = False
    results: Dict[str, Any] = {}
    artifacts: List[str] = []
```

#### 4. Agent Result
```python
class AgentResult(BaseModel):
    agent_id: str
    timestamp: datetime
    result_data: Any
    artifacts: List[str] = []
    token_usage: Dict[str, int] = {}
    duration_ms: int
```

### Specialized Models

#### 1. Story Model
```python
class StoryDependency(BaseModel):
    story_id: str
    dependency_type: str
    description: str

class StoryContext(BaseModel):
    project_overview: str
    current_state: str
    purpose: str
    position: str

class StorySpecs(BaseModel):
    inputs: List[Dict[str, Any]]
    outputs: List[Dict[str, Any]]
    business_rules: List[str]
    data_models: List[Dict[str, Any]]

class StoryDetails(BaseModel):
    file_locations: List[str]
    project_structure: str
    design_patterns: List[str]
    code_standards: List[str]
    framework_guidelines: List[str]

class Story(BaseModel):
    id: str
    title: str
    epic_id: str
    context: StoryContext
    dependencies: List[StoryDependency]
    technical_specs: StorySpecs
    implementation_details: StoryDetails
    integration_points: Dict[str, Any]
    acceptance_criteria: List[str]
    testing_guidelines: Dict[str, Any]
    additional_resources: Dict[str, Any]
```

#### 2. Technical Standards Model
```python
class CodeStandard(BaseModel):
    language: str
    conventions: Dict[str, Any]
    patterns: List[str]
    linting_rules: Dict[str, Any]

class CrossCuttingConcern(BaseModel):
    name: str
    requirements: List[str]
    implementation_guidelines: str
    exceptions: List[str]

class TechnicalStandards(BaseModel):
    version: str
    code_standards: Dict[str, CodeStandard]
    cross_cutting_concerns: Dict[str, CrossCuttingConcern]
    approved_patterns: Dict[str, str]
    anti_patterns: Dict[str, str]
    integration_standards: Dict[str, Any]
```

## Agent Architecture

### Agent Framework

The agent architecture leverages Pydantic AI for structure and type safety:

```
┌───────────────────────────────────────────────────────────┐
│                  Pydantic AI Agent                        │
├───────────────┬───────────────────────┬───────────────────┤
│ System Prompts│ Function Tools        │ Result Type       │
├───────────────┼───────────────────────┼───────────────────┤
│ Dependencies  │ Result Validators     │ Message History   │
└───────────────┴───────────────────────┴───────────────────┘
```

### Agent Specializations

The system includes these specialized agents:

1. **Business Analyst Agent**
   - Transforms ideas into project vision
   - Specializations: Market analysis, feature specification, scope management

2. **Project Manager Agent**
   - Researches and creates requirements
   - Specializations: Risk analysis, timeline estimation, competitive analysis, compliance

3. **Architect Agent**
   - Designs technical architecture
   - Specializations: Frontend, backend, database, security, performance, infrastructure

4. **UX/UI Designer Agent**
   - Creates interface designs
   - Specializations: Wireframing, user flows, design systems, accessibility

5. **Product Owner Agent**
   - Creates task lists and implementation sequences
   - Specializations: Dependency mapping, implementation sequencing, effort estimation

6. **Scrum Master Agent**
   - Breaks down epics and stories
   - Specializations: Epic coordination, story refinement, acceptance criteria

7. **Technical Standards Coordinator**
   - Creates and maintains technical standards
   - Specializations: Coding standards, architecture patterns, cross-cutting concerns

8. **Quality Assurance Agent**
   - Creates test plans and quality criteria
   - Specializations: Unit testing, integration testing, performance testing

9. **DevOps Specialist Agent**
   - Designs infrastructure and deployment
   - Specializations: CI/CD, infrastructure configuration, monitoring

### ADK-Inspired Workflow Patterns

Extended with workflow patterns inspired by ADK:

1. **SequentialAgent**
   - Executes agents in sequence
   - Passes context between agents
   - Combines results

2. **ParallelAgent**
   - Executes agents concurrently
   - Aggregates results
   - Manages resources

3. **LoopAgent**
   - Repeatedly executes agents
   - Checks termination conditions
   - Maintains state between iterations

### Agent Delegation System

Enables agents to transfer control to specialized agents:

1. **AgentRegistry**
   - Maintains registry of available agents
   - Provides agent lookup and discovery

2. **DelegationManager**
   - Handles delegation requests
   - Manages context passing
   - Tracks delegation history

3. **Transfer Utility**
   - Simplifies delegation syntax
   - Provides error handling
   - Manages state transitions

## Workflow Architecture

### Workflow Components

The workflow system is built on Pydantic Graph:

1. **Graph Nodes**
   - Represent workflow states
   - Define transition logic
   - Access and modify shared state

2. **Workflow State**
   - Maintains workflow context
   - Stores intermediate results
   - Tracks progress and history

3. **Transitions**
   - Define possible next states
   - Implement conditional logic
   - Handle error cases

### Core Workflows

Using Pydantic Graph, the system implements these primary workflows:

1. **Project Initialization Workflow**
   - Creates project structure
   - Generates project vision
   - Sets up initial configuration

2. **Requirements Gathering Workflow**
   - Analyzes project vision
   - Gathers functional and non-functional requirements
   - Identifies constraints and assumptions

3. **Architecture Design Workflow**
   - Evaluates technology options
   - Designs system architecture
   - Defines component relationships

4. **Technical Standards Workflow**
   - Defines coding standards
   - Establishes cross-cutting concerns
   - Creates reusable patterns

5. **Task List Workflow**
   - Breaks down project into tasks
   - Identifies dependencies
   - Creates implementation sequence

6. **Story Creation Workflow**
   - Organizes tasks into epics
   - Creates detailed stories
   - Validates against readiness checklist

7. **Documentation Export Workflow**
   - Generates complete documentation package
   - Creates cross-references
   - Formats for target consumers

## User Interface Design

### Command Line Interface

The CLI is implemented using Typer with Rich for formatting:

1. **Core Commands**
   - `new`: Create a new project
   - `continue`: Continue working on an existing project
   - `agents`: List and manage agents
   - `export`: Export documentation

2. **Interactive Commands**
   - `brainstorm`: Start interactive brainstorming session
   - `decide`: Present architecture/design decisions
   - `review`: Review and validate documentation

3. **Utility Commands**
   - `theme`: Manage UI themes
   - `plugins`: Manage plugins
   - `config`: Configure system settings

### Terminal UI

Rich components enhance the terminal interface:

1. **Panels**
   - Content panels with borders
   - Collapsible sections
   - Nested content

2. **Tables**
   - Data presentation
   - Comparison views
   - Sortable columns

3. **Progress Indicators**
   - Progress bars
   - Spinners
   - Task completion indicators

4. **Interactive Elements**
   - Selection menus
   - Confirmation prompts
   - Multi-step wizards

### Theme System

UI appearance is customized through themes:

1. **Theme Components**
   - Color definitions
   - Style settings
   - Typography rules

2. **Theme Types**
   - Light theme
   - Dark theme
   - High-contrast theme
   - Custom themes

## File Structure

The project follows this directory structure:

```
projectplanner/
├── src/
│   ├── cli/                    # CLI commands and interface
│   │   ├── commands/           # Command implementations
│   │   ├── formatters/         # Output formatters
│   │   ├── prompts/            # User input handlers
│   │   └── theming/            # Theme management
│   ├── core/                   # Core engine
│   │   ├── config.py           # Configuration management
│   │   ├── engine.py           # Central engine
│   │   └── errors.py           # Error handling
│   ├── agents/                 # Agent implementations
│   │   ├── manager.py          # Agent manager
│   │   ├── base.py             # Base agent classes
│   │   ├── business_analyst.py # Business Analyst agent
│   │   ├── project_manager.py  # Project Manager agent
│   │   └── ...                 # Other agent implementations
│   ├── workflow/               # Workflow system
│   │   ├── engine.py           # Workflow engine
│   │   ├── nodes.py            # Node implementations
│   │   ├── transitions.py      # Transition logic
│   │   └── ...                 # Workflow definitions
│   ├── document_store/         # Document management
│   │   ├── store.py            # Document store
│   │   ├── models.py           # Document models
│   │   └── validators.py       # Document validators
│   ├── file_manager/           # File operations
│   │   ├── manager.py          # File manager
│   │   └── paths.py            # Path utilities
│   ├── template_engine/        # Template handling
│   │   ├── engine.py           # Template engine
│   │   ├── filters.py          # Template filters
│   │   └── loaders.py          # Template loaders
│   ├── plugins/                # Plugin system
│   │   ├── manager.py          # Plugin manager
│   │   ├── base.py             # Base plugin class
│   │   └── discovery.py        # Plugin discovery
│   └── utils/                  # Utility functions
│       ├── logging.py          # Logging utilities
│       ├── formatting.py       # Text formatting
│       └── validation.py       # Data validation
├── templates/                  # Document templates
│   ├── project_vision/         # Vision templates
│   ├── requirements/           # Requirements templates
│   ├── architecture/           # Architecture templates
│   ├── story/                  # Story templates
│   └── ...                     # Other templates
├── tests/                      # Tests
│   ├── unit/                   # Unit tests
│   ├── integration/            # Integration tests
│   └── fixtures/               # Test fixtures
├── docs/                       # Documentation
│   ├── api/                    # API documentation
│   ├── user/                   # User guides
│   └── dev/                    # Developer documentation
├── pyproject.toml              # Project configuration
└── README.md                   # Project overview
```

## External Interfaces

### LLM Provider Integration

The system supports multiple LLM providers through Pydantic AI:

1. **OpenAI**
   - GPT-4o model support
   - API key configuration
   - Token tracking

2. **Anthropic**
   - Claude models support
   - API key configuration
   - Token tracking

3. **Local Models**
   - Support for local LLM deployment
   - Configuration for endpoint URL
   - Resource management

### Plugin API

Plugins extend system functionality through defined interfaces:

1. **Plugin Types**
   - Agent plugins
   - Workflow plugins
   - Template plugins
   - Integration plugins

2. **Plugin Interface**
   - Initialization
   - Command registration
   - Agent registration
   - Workflow registration

## Security Considerations

1. **API Key Management**
   - Secure storage of API keys
   - Environment variable integration
   - Key rotation support

2. **File Security**
   - Permissions management
   - Sensitive data handling
   - Temporary file cleanup

3. **Plugin Security**
   - Plugin verification
   - Sandboxed execution
   - Resource limitations

## Performance Considerations

1. **Optimization Strategies**
   - Context window management
   - Document chunking
   - Caching for repeated operations

2. **Resource Management**
   - Token usage tracking
   - Concurrent request limiting
   - Memory usage optimization

3. **Response Time**
   - Streaming for long-running operations
   - Progress indicators
   - Background processing

## Deployment Options

1. **Local Installation**
   - PyPI package installation
   - Direct from source
   - Development setup

2. **Containerized Deployment**
   - Docker container
   - Docker Compose configuration
   - Environment configuration

3. **CI/CD Integration**
   - GitHub Actions integration
   - Automated testing
   - Release management

## Extension Points

The architecture provides these key extension points:

1. **Agent Extensions**
   - Custom agent implementations
   - Tool extensions
   - Result type extensions

2. **Workflow Extensions**
   - Custom node types
   - New workflow definitions
   - Transition extensions

3. **Template Extensions**
   - Custom template types
   - Template filters
   - Rendering extensions

4. **UI Extensions**
   - Custom commands
   - Output formatters
   - Theme extensions

## Technical Decisions

### Key Technology Choices

1. **Python 3.10+**
   - Modern language features
   - Type annotations
   - Pattern matching

2. **Pydantic AI**
   - Type-safe agent definitions
   - Structured result validation
   - Model provider abstraction

3. **Typer + Rich**
   - Modern CLI interface
   - Rich formatting
   - Interactive elements

4. **Jinja2**
   - Powerful templating
   - Template inheritance
   - Filter extensibility

### Alternative Considerations

1. **Custom Agent Framework vs. Pydantic AI**
   - Pydantic AI chosen for type safety and validation
   - Reduces boilerplate code
   - Provides model provider abstraction

2. **Web UI vs. Terminal UI**
   - Terminal UI chosen for simplicity
   - Reduces dependencies
   - Better fits CLI workflow

3. **SQLite vs. File-based Storage**
   - File-based storage chosen for portability
   - Simplifies installation
   - Better fits document-oriented data

## Future Considerations

1. **Potential Enhancements**
   - Web UI for broader accessibility
   - Database backend for larger projects
   - Collaborative editing support

2. **Integration Opportunities**
   - Git integration for version control
   - CI/CD pipeline integration
   - Issue tracker integration

3. **Scaling Strategies**
   - Distributed agent execution
   - Cloud storage integration
   - Enterprise authentication