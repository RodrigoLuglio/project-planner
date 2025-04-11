# ProjectPlanner - Agent Design Specification

This document provides detailed specifications for the AI agents in ProjectPlanner, including their responsibilities, implementation details, system prompts, and tools.

## Agent Architecture Overview

All agents in ProjectPlanner are built on Pydantic AI with a common architecture:

```
┌───────────────────────────────────────────────────────────┐
│                  Pydantic AI Agent                        │
├───────────────┬───────────────────────┬───────────────────┤
│ System Prompts│ Function Tools        │ Result Type       │
├───────────────┼───────────────────────┼───────────────────┤
│ Dependencies  │ Result Validators     │ Message History   │
└───────────────┴───────────────────────┴───────────────────┘
```

### Common Components

All agents share these elements:

1. **System Prompts**
   - Static role definition prompts
   - Dynamic context-aware prompts
   - Project-specific information

2. **Function Tools**
   - Document retrieval tools
   - Search capabilities
   - Cross-referencing tools
   - Specialized utilities

3. **Result Types**
   - Pydantic models for validation
   - Structured output formats
   - Custom validators

4. **Dependencies**
   - Document store access
   - File manager access
   - Template engine access
   - Agent registry access

5. **Message History**
   - Conversation context
   - Previous interactions
   - Context prioritization

## ADK-Inspired Workflow Agents

The system extends the base agent architecture with ADK-inspired workflow agents:

### Base Workflow Agent

```python
class WorkflowAgent(Agent, Generic[T], ABC):
    """Base class for workflow agent patterns."""
    
    name: str
    description: str
    result_type: Type[T]
    
    async def orchestrate(self, ctx: RunContext) -> T:
        """Execute the workflow orchestration logic."""
        pass
        
    async def run(self, ctx: RunContext) -> T:
        """Main entry point for workflow execution."""
        result = await self.orchestrate(ctx)
        return result
```

### Workflow Agent Types

The system implements these workflow agent patterns:

1. **SequentialAgent**
   - Executes sub-agents in sequence
   - Passes context between agents
   - Combines results

2. **ParallelAgent**
   - Executes sub-agents concurrently
   - Aggregates results
   - Manages resource allocation

3. **LoopAgent**
   - Repeatedly executes sub-agents
   - Checks termination conditions
   - Maintains state between iterations

## Agent Delegation System

The system implements agent delegation through these components:

### Agent Registry

```python
class AgentRegistry:
    """Manages available agents and handles delegation."""
    
    def __init__(self):
        self.agents: Dict[str, Agent] = {}
        
    def register(self, agent: Agent) -> None:
        """Register an agent in the system."""
        agent_name = getattr(agent, "name", None)
        if not agent_name:
            raise ValueError("Agent must have a name to be registered")
        
        self.agents[agent_name] = agent
        
    def get_agent(self, name: str) -> Optional[Agent]:
        """Get agent by name."""
        return self.agents.get(name)
        
    def list_agents(self) -> Dict[str, str]:
        """List all registered agents with their descriptions."""
        return {
            name: getattr(agent, "description", "No description") 
            for name, agent in self.agents.items()
        }
```

### Delegation Manager

```python
@dataclass
class DelegationRequest:
    """Represents a request to delegate to another agent."""
    target_agent: str
    input_data: Optional[Dict[str, Any]] = None
    metadata: Dict[str, Any] = field(default_factory=dict)

class DelegationManager:
    """Manages the delegation of tasks between agents."""
    
    def __init__(self, registry: AgentRegistry):
        self.registry = registry
        
    async def delegate(
        self, 
        request: DelegationRequest, 
        ctx: RunContext
    ) -> Any:
        """Execute delegation to the target agent."""
        # Implementation details...
```

### Transfer Utility

```python
async def transfer_to_agent(
    ctx: RunContext, 
    target_agent: str, 
    input_data: Optional[Dict[str, Any]] = None
) -> Any:
    """Transfer control to another agent (simplified delegation)."""
    request = DelegationRequest(
        target_agent=target_agent,
        input_data=input_data
    )
    
    return await ctx.deps.delegation_manager.delegate(request, ctx)
```

## Specialized Agents

### 1. Business Analyst Agent

**Responsibility**: Transforms abstract ideas into clear project vision.

#### Result Type

```python
class ProjectVision(BaseModel):
    """Output from the Business Analyst."""
    problem_statement: str
    proposed_solution: List[str]
    key_stakeholders: List[Dict[str, str]]
    success_criteria: List[str]
    vision_document: str
```

#### System Prompt

```
You are an expert Business Analyst with extensive experience in software development projects. 
Your task is to help transform initial ideas into clear, actionable project visions.

You have expertise in:
- Identifying core problems and needs
- Defining solutions that address those problems
- Identifying key stakeholders and their interests
- Establishing clear success criteria
- Communicating complex concepts clearly

When analyzing project ideas:
1. First understand the core problem being solved
2. Consider various solution approaches
3. Identify who will be impacted by the solution
4. Define how success will be measured
5. Document everything clearly and concisely
```

#### Dynamic Context

```python
@business_analyst.system_prompt
def add_project_context(ctx: RunContext[ProjectDependencies]) -> str:
    """Add relevant project context to the system prompt."""
    return f"Project Name: {ctx.deps.document_store.get_project_name()}\n" + \
           f"Current Stage: {ctx.deps.document_store.get_project_stage()}\n"
```

#### Tools

1. **search_similar_projects**
   ```python
   @business_analyst.tool
   async def search_similar_projects(ctx: RunContext[ProjectDependencies], keywords: List[str]) -> List[Dict]:
       """Search for similar projects in our knowledge base."""
       return await ctx.deps.document_store.search_projects(keywords)
   ```

2. **analyze_market_viability**
   ```python
   @business_analyst.tool
   async def analyze_market_viability(ctx: RunContext[ProjectDependencies], concept: str) -> Dict:
       """Analyze market viability of a concept."""
       result = await transfer_to_agent(ctx, "market_analyst", {"concept": concept})
       return result
   ```

3. **generate_vision_document**
   ```python
   @business_analyst.tool
   async def generate_vision_document(ctx: RunContext[ProjectDependencies], vision_data: Dict) -> str:
       """Generate a formatted vision document from data."""
       template = await ctx.deps.template_engine.get_template("project_vision")
       return await template.render(vision_data)
   ```

### 2. Project Manager Agent

**Responsibility**: Researches and creates comprehensive requirements.

#### Result Type

```python
class RequirementItem(BaseModel):
    id: str
    description: str
    priority: str
    source: str
    notes: Optional[str] = None

class PRDDocument(BaseModel):
    """Product Requirements Document from Project Manager."""
    functional_requirements: List[RequirementItem]
    non_functional_requirements: List[RequirementItem]
    constraints: List[str]
    assumptions: List[str]
    requirements_document: str
```

#### System Prompt

```
You are an expert Project Manager specializing in requirements gathering and analysis.
Your task is to create comprehensive requirement documents that clearly define what a 
project must accomplish.

You have expertise in:
- Gathering and analyzing functional requirements
- Identifying non-functional requirements
- Recognizing project constraints and assumptions
- Organizing requirements logically and clearly
- Prioritizing requirements based on business value

When creating requirements:
1. Ensure each requirement is clear, specific, and testable
2. Distinguish between must-have and nice-to-have features
3. Identify technical and business constraints
4. Document underlying assumptions
5. Organize requirements in a logical structure
```

#### Tools

1. **analyze_requirements**
   ```python
   @project_manager.tool
   async def analyze_requirements(ctx: RunContext[ProjectDependencies], vision_document: str) -> Dict:
       """Analyze a vision document to extract requirements."""
       # Implementation details...
   ```

2. **identify_risks**
   ```python
   @project_manager.tool
   async def identify_risks(ctx: RunContext[ProjectDependencies], requirements: List[Dict]) -> List[Dict]:
       """Identify potential risks based on requirements."""
       result = await transfer_to_agent(ctx, "risk_analyst", {"requirements": requirements})
       return result
   ```

3. **generate_requirements_document**
   ```python
   @project_manager.tool
   async def generate_requirements_document(ctx: RunContext[ProjectDependencies], req_data: Dict) -> str:
       """Generate a formatted requirements document."""
       template = await ctx.deps.template_engine.get_template("requirements")
       return await template.render(req_data)
   ```

### 3. Architect Agent

**Responsibility**: Designs technical architecture based on requirements.

#### Result Type

```python
class TechStackItem(BaseModel):
    name: str
    purpose: str
    alternatives: List[str]
    justification: str

class Component(BaseModel):
    name: str
    responsibility: str
    interfaces: List[str]
    dependencies: List[str]

class ArchitectureDocument(BaseModel):
    """Architecture document from Architect."""
    arch_type: str
    tech_stack: List[TechStackItem]
    components: List[Component]
    data_models: List[Dict[str, Any]]
    integration_points: List[Dict[str, str]]
    architecture_document: str
```

#### System Prompt

```
You are an expert Software Architect responsible for designing robust, scalable 
technical architectures based on project requirements.

You have expertise in:
- Selecting appropriate architectural patterns
- Choosing technology stacks based on requirements
- Designing components and their interactions
- Creating data models and schemas
- Identifying integration points and interfaces

When designing architecture:
1. First understand the functional and non-functional requirements
2. Consider multiple architectural approaches before selecting one
3. Justify technology choices with clear reasoning
4. Design components with single responsibilities
5. Define clear interfaces between components
6. Consider security, performance, and scalability
```

#### Tools

1. **evaluate_tech_stack**
   ```python
   @architect.tool
   async def evaluate_tech_stack(ctx: RunContext[ProjectDependencies], requirements: Dict) -> List[Dict]:
       """Evaluate and recommend technology stack options."""
       # Implementation details...
   ```

2. **design_components**
   ```python
   @architect.tool
   async def design_components(ctx: RunContext[ProjectDependencies], arch_type: str, requirements: Dict) -> List[Dict]:
       """Design system components based on architecture type and requirements."""
       # Implementation details...
   ```

3. **generate_architecture_document**
   ```python
   @architect.tool
   async def generate_architecture_document(ctx: RunContext[ProjectDependencies], arch_data: Dict) -> str:
       """Generate a formatted architecture document."""
       template = await ctx.deps.template_engine.get_template("architecture")
       return await template.render(arch_data)
   ```

### 4. UX/UI Designer Agent

**Responsibility**: Creates interface designs and user flows.

#### Result Type

```python
class UserFlow(BaseModel):
    name: str
    steps: List[Dict[str, str]]
    notes: str

class DesignSystem(BaseModel):
    colors: Dict[str, str]
    typography: Dict[str, Any]
    components: List[Dict[str, Any]]

class DesignDocument(BaseModel):
    """Design document from UX/UI Designer."""
    user_flows: List[UserFlow]
    wireframes: List[str]
    design_system: DesignSystem
    accessibility_guidelines: List[str]
    design_document: str
```

#### System Prompt

```
You are an expert UX/UI Designer responsible for creating intuitive, 
accessible user interfaces and experiences.

You have expertise in:
- Creating user flows that guide users through tasks
- Designing wireframes that visualize interfaces
- Establishing consistent design systems
- Ensuring accessibility for all users
- Balancing aesthetics with functionality

When designing interfaces:
1. First understand the user needs and journey
2. Create logical flows based on user goals
3. Design consistent, intuitive interfaces
4. Establish clear design patterns for reusability
5. Ensure accessibility is built into all designs
```

### 5. Product Owner Agent

**Responsibility**: Creates task lists with dependencies and implementation sequences.

#### Result Type

```python
class Task(BaseModel):
    id: str
    title: str
    description: str
    prerequisites: List[str]
    effort: str
    priority: str

class TaskList(BaseModel):
    """Task list from Product Owner."""
    tasks: List[Task]
    dependencies: Dict[str, List[str]]
    implementation_sequence: List[str]
    task_list_document: str
```

#### System Prompt

```
You are an expert Product Owner responsible for breaking down projects
into implementable tasks and establishing optimal implementation sequences.

You have expertise in:
- Breaking down complex requirements into discrete tasks
- Identifying dependencies between tasks
- Prioritizing tasks based on business value and technical constraints
- Creating logical implementation sequences
- Estimating relative effort for tasks

When creating task lists:
1. Ensure tasks are granular enough to be implemented independently
2. Identify all prerequisites for each task
3. Create a logical sequence that respects dependencies
4. Prioritize tasks that unlock the most subsequent work
5. Consider business value when establishing priorities
```

### 6. Scrum Master Agent

**Responsibility**: Breaks down tasks into implementation-ready stories.

#### Result Type

```python
class Epic(BaseModel):
    id: str
    title: str
    description: str
    stories: List[str]

class StoryDefinition(BaseModel):
    id: str
    title: str
    epic_id: str
    description: str
    acceptance_criteria: List[str]
    technical_details: Dict[str, Any]

class EpicStories(BaseModel):
    """Epics and stories from Scrum Master."""
    epics: List[Epic]
    stories: List[StoryDefinition]
    stories_document: str
```

#### System Prompt

```
You are an expert Scrum Master responsible for organizing project work into
well-defined epics and stories that are ready for implementation.

You have expertise in:
- Grouping related tasks into logical epics
- Creating detailed, self-contained stories
- Defining clear acceptance criteria
- Ensuring all implementation details are specified
- Validating story readiness for development

When creating stories:
1. Group related functionality into cohesive epics
2. Make each story self-contained and independently implementable
3. Include all technical details needed for implementation
4. Define specific, testable acceptance criteria
5. Validate stories against the readiness checklist
```

### 7. Technical Standards Coordinator

**Responsibility**: Creates and maintains technical standards documentation.

#### Result Type

```python
class CodeStandard(BaseModel):
    language: str
    conventions: Dict[str, Any]
    linting_rules: Dict[str, Any]

class DesignPattern(BaseModel):
    name: str
    purpose: str
    implementation: str
    when_to_use: str

class TechStandards(BaseModel):
    """Technical standards from Technical Standards Coordinator."""
    code_standards: Dict[str, CodeStandard]
    design_patterns: List[DesignPattern]
    cross_cutting_concerns: Dict[str, List[str]]
    naming_conventions: Dict[str, str]
    standards_document: str
```

#### System Prompt

```
You are an expert Technical Standards Coordinator responsible for establishing
consistent standards, patterns, and practices across a software project.

You have expertise in:
- Defining coding standards for multiple languages
- Establishing design patterns and when to use them
- Creating guidelines for cross-cutting concerns
- Defining naming conventions and structural patterns
- Ensuring technical consistency across a project

When creating technical standards:
1. Create clear, specific guidelines that are easy to follow
2. Provide examples for each standard or pattern
3. Explain the reasoning behind each standard
4. Address all cross-cutting concerns systematically
5. Create standards that balance flexibility with consistency
```

### 8. Quality Assurance Agent

**Responsibility**: Creates test plans and quality criteria.

#### Result Type

```python
class TestCase(BaseModel):
    id: str
    description: str
    steps: List[str]
    expected_result: str
    category: str

class QualityMetric(BaseModel):
    name: str
    description: str
    target: str
    measurement_method: str

class TestPlan(BaseModel):
    """Test plan from Quality Assurance Agent."""
    test_strategy: str
    test_cases: List[TestCase]
    quality_metrics: List[QualityMetric]
    test_environment: Dict[str, Any]
    test_plan_document: str
```

#### System Prompt

```
You are an expert Quality Assurance Specialist responsible for creating
comprehensive test plans and defining quality criteria for software projects.

You have expertise in:
- Developing test strategies and plans
- Creating detailed test cases
- Defining quality metrics and targets
- Designing test environments and data
- Identifying edge cases and boundary conditions

When creating test plans:
1. Develop a comprehensive testing strategy
2. Create detailed test cases with clear steps
3. Define measurable quality metrics
4. Identify potential edge cases and boundary conditions
5. Specify test data and environment requirements
```

### 9. DevOps Specialist Agent

**Responsibility**: Designs deployment pipelines and infrastructure.

#### Result Type

```python
class Infrastructure(BaseModel):
    components: List[Dict[str, Any]]
    configuration: Dict[str, Any]
    scaling: Dict[str, Any]

class CICDPipeline(BaseModel):
    stages: List[Dict[str, Any]]
    triggers: List[str]
    environments: List[str]

class DevOpsDocument(BaseModel):
    """DevOps document from DevOps Specialist."""
    deployment_strategy: str
    infrastructure: Infrastructure
    ci_cd_pipeline: CICDPipeline
    monitoring: Dict[str, Any]
    security: List[str]
    devops_document: str
```

#### System Prompt

```
You are an expert DevOps Specialist responsible for designing deployment
pipelines, infrastructure configurations, and operational procedures.

You have expertise in:
- Designing infrastructure architectures
- Creating CI/CD pipelines
- Configuring monitoring and alerting
- Implementing security best practices
- Automating operational procedures

When creating DevOps documentation:
1. Design scalable, reliable infrastructure
2. Create efficient CI/CD pipelines for automated deployment
3. Configure comprehensive monitoring and alerting
4. Implement robust security measures
5. Document operational procedures clearly
```

## Implementation Approach

### Agent Creation Flow

1. **Configuration Loading**
   - Load agent configuration
   - Configure system prompts
   - Set up result type

2. **Tool Registration**
   - Register common tools
   - Register specialized tools
   - Configure tool access control

3. **Dependency Injection**
   - Set up document store access
   - Configure file manager access
   - Set up template engine access

4. **Agent Registration**
   - Register with agent registry
   - Configure delegation access
   - Set up sub-agent relationships

### Agent Execution Pattern

1. **Context Preparation**
   - Load project context
   - Prepare necessary dependencies
   - Set up message history

2. **Agent Execution**
   - Run agent with prompt
   - Track token usage
   - Handle potential errors

3. **Result Processing**
   - Validate result structure
   - Process and store artifacts
   - Update project state

4. **Next Steps**
   - Determine next workflow steps
   - Update workflow state
   - Trigger notifications

## Testing Strategy

### Agent Testing

1. **Unit Testing**
   - Test individual tools
   - Validate system prompts
   - Check result validation

2. **Behavioral Testing**
   - Test agent responses to inputs
   - Validate decision making
   - Check error handling

3. **Mock Testing**
   - Use TestModel for deterministic responses
   - Test tool calling patterns
   - Validate result parsing

### Integration Testing

1. **Agent Interaction Testing**
   - Test delegation between agents
   - Validate context passing
   - Check result integration

2. **Workflow Testing**
   - Test complete workflows
   - Validate state transitions
   - Check workflow outputs

3. **End-to-End Testing**
   - Test complete user scenarios
   - Validate document generation
   - Check project progression

## Performance Considerations

### Context Management

1. **Window Optimization**
   - Prioritize relevant context
   - Chunk large documents
   - Summarize historical interactions

2. **Token Efficiency**
   - Use concise system prompts
   - Optimize tool inputs and outputs
   - Implement token budget management

### Caching Strategy

1. **Result Caching**
   - Cache frequently used results
   - Implement cache invalidation
   - Use cache warming for common operations

2. **Document Caching**
   - Cache document sections
   - Implement partial updates
   - Use lazy loading

## Error Handling

### Agent-Specific Errors

1. **Model Errors**
   - Handle unavailable models
   - Manage rate limiting
   - Implement fallback strategies

2. **Tool Errors**
   - Handle tool execution failures
   - Implement retry logic
   - Provide helpful error information

3. **Result Validation**
   - Handle invalid results
   - Implement fallback strategies
   - Provide debugging information

## Workflow Error Recovery

### Error Recovery Strategies

1. **Transient Errors**
   - Implement automatic retry with exponential backoff
   - Detect rate limiting and enforce appropriate waiting periods
   - Fall back to alternative models when appropriate

2. **Content Errors**
   - Implement result validators for structural errors
   - Add content validators for semantic errors
   - Create repair strategies for fixable errors

3. **Context Errors**
   - Detect and handle context window overflows
   - Implement chunking for large inputs
   - Use summarization for history compression

### Recovery Implementation

```python
class AgentWorkflowRecovery:
    """Implements recovery strategies for agent workflow errors."""
    
    def __init__(self, max_retries: int = 3, fallback_models: Dict[str, str] = None):
        self.max_retries = max_retries
        self.fallback_models = fallback_models or {}
        
    async def execute_with_recovery(
        self, 
        agent: Agent, 
        prompt: str, 
        ctx: RunContext
    ) -> Any:
        """Execute an agent with error recovery."""
        retries = 0
        last_error = None
        
        while retries <= self.max_retries:
            try:
                return await agent.run(prompt, ctx)
            except ModelRateLimitError as e:
                # Handle rate limiting
                retry_after = getattr(e, "retry_after", retries * 2 + 1)
                logger.warning(f"Rate limit hit, waiting {retry_after}s before retry")
                await asyncio.sleep(retry_after)
                retries += 1
                last_error = e
            except ModelConnectionError as e:
                # Try fallback model if available
                original_model = agent.model
                if original_model in self.fallback_models and retries < len(self.fallback_models[original_model]):
                    fallback = self.fallback_models[original_model][retries]
                    logger.warning(f"Connection error with {original_model}, trying fallback {fallback}")
                    with agent.override(model=fallback):
                        try:
                            return await agent.run(prompt, ctx)
                        except Exception as fallback_error:
                            logger.error(f"Fallback model {fallback} also failed: {fallback_error}")
                retries += 1
                last_error = e
            except ValidationError as e:
                # Try to repair the result if possible
                if retries < self.max_retries:
                    repair_prompt = self._create_repair_prompt(e, prompt)
                    logger.warning(f"Validation error, attempting repair: {e}")
                    try:
                        return await agent.run(repair_prompt, ctx)
                    except Exception as repair_error:
                        logger.error(f"Repair attempt failed: {repair_error}")
                retries += 1
                last_error = e
            except Exception as e:
                # Handle other exceptions
                logger.error(f"Unexpected error in agent execution: {e}")
                retries += 1
                last_error = e
                
        # If we've exhausted retries, raise the last error
        raise AgentExecutionError(f"Failed after {retries} retries: {last_error}") from last_error
        
    def _create_repair_prompt(self, validation_error: ValidationError, original_prompt: str) -> str:
        """Create a prompt to repair validation errors."""
        error_details = str(validation_error)
        return f"""
Your previous response had validation errors. Please fix the following issues and respond again:

ERROR DETAILS:
{error_details}

ORIGINAL PROMPT:
{original_prompt}

Please provide a corrected response that fixes these validation issues.
"""
```

## User Interaction Models

### Conversation Management

1. **Single-Turn Interactions**
   - Simple question/answer pattern
   - Process prompt and return formatted response
   - Validate input and output

2. **Multi-Turn Conversations**
   - Maintain conversation history
   - Track conversation state
   - Implement dialogue management

3. **Contextual Responses**
   - Incorporate previous messages
   - Reference past outputs
   - Build on established context

### Streaming Response Handling

1. **Stream Processing**
   - Implement token-by-token streaming
   - Process partial outputs
   - Handle streaming errors

2. **Interactive Streaming**
   - Detect questions requiring responses
   - Pause for user input
   - Resume with additional context

3. **Progress Indicators**
   - Show thinking/processing indicators
   - Display incremental progress
   - Provide estimated completion times

### Conversation Implementation

```python
class AgentConversationManager:
    """Manages conversational interactions with agents."""
    
    def __init__(self, agent: Agent, ctx: RunContext):
        self.agent = agent
        self.ctx = ctx
        self.message_history = []
        
    async def single_turn(self, prompt: str) -> str:
        """Process a single-turn interaction."""
        try:
            result = await self.agent.run(prompt, self.ctx)
            return self._format_response(result)
        except Exception as e:
            logger.error(f"Error in single-turn interaction: {e}")
            return f"Error: {str(e)}"
    
    async def start_conversation(self, initial_prompt: str):
        """Start a multi-turn conversation."""
        self.message_history = []
        return await self.continue_conversation(initial_prompt)
    
    async def continue_conversation(self, user_input: str) -> str:
        """Continue an existing conversation."""
        # Add user message to history
        self.message_history.append({"role": "user", "content": user_input})
        
        try:
            result = await self.agent.run(
                user_input, 
                self.ctx,
                message_history=self.message_history
            )
            
            # Add agent response to history
            response = self._format_response(result)
            self.message_history.append({"role": "assistant", "content": response})
            
            return response
        except Exception as e:
            logger.error(f"Error in conversation: {e}")
            error_msg = f"Error: {str(e)}"
            self.message_history.append({"role": "assistant", "content": error_msg})
            return error_msg
    
    async def interactive_stream(self, initial_prompt: str, console: Console = None):
        """Start an interactive streaming session."""
        console = console or Console()
        self.message_history = []
        
        # Add user message to history
        self.message_history.append({"role": "user", "content": initial_prompt})
        
        async with self.agent.run_stream(
            initial_prompt,
            self.ctx,
            message_history=self.message_history
        ) as stream:
            # Stream the response
            console.print(f"\n[bold blue]{self.agent.name}:[/bold blue]", end=" ")
            
            accumulated_response = ""
            question_detected = False
            
            async for chunk in stream.stream_text():
                console.print(chunk, end="")
                accumulated_response += chunk
                
                # Check if the chunk contains a question requiring user input
                if self._contains_question(accumulated_response) and not question_detected:
                    question_detected = True
                    console.print("\n\n[bold green]You:[/bold green] ", end="")
                    user_input = await self._get_async_input()
                    
                    # Send additional input to the stream
                    await stream.send_additional_input(user_input)
                    
                    # Reset for potential additional questions
                    question_detected = False
                    accumulated_response = ""
            
            # Add full response to history
            self.message_history.append({
                "role": "assistant", 
                "content": accumulated_response
            })
            
            return accumulated_response
    
    def _format_response(self, result: Any) -> str:
        """Format the agent result for user display."""
        if hasattr(result, "data"):
            # For Pydantic model results
            if hasattr(result.data, "response"):
                return result.data.response
            # Default to string representation
            return str(result.data)
        # For string or other results
        return str(result)
    
    def _contains_question(self, text: str) -> bool:
        """Detect if text contains a question requiring user input."""
        # Simple heuristic: check for question marks followed by a new line
        # Can be enhanced with more sophisticated NLP
        question_patterns = [
            r'\?\s*,  # Question mark at the end
            r'\?\s*\n',  # Question mark followed by newline
            r'Please provide|Can you specify|What is your preference',  # Common question phrases
        ]
        
        for pattern in question_patterns:
            if re.search(pattern, text):
                return True
        return False
    
    async def _get_async_input(self) -> str:
        """Get user input asynchronously."""
        loop = asyncio.get_running_loop()
        return await loop.run_in_executor(None, input)
```

## Agent Evaluation and Quality

### Quality Metrics

1. **Accuracy Metrics**
   - Output validation rate
   - Document completeness score
   - Technical correctness assessment

2. **Usefulness Metrics**
   - Actionability of outputs
   - Implementation readiness
   - User satisfaction ratings

3. **Efficiency Metrics**
   - Token usage
   - Response time
   - Completion rate

### Evaluation Framework

1. **Self-Evaluation**
   - Agent reflection on outputs
   - Quality assurance checks
   - Self-correction mechanisms

2. **Cross-Evaluation**
   - Peer review by other agents
   - Consistency validation
   - Implementation feasibility check

3. **User Feedback**
   - Explicit ratings collection
   - Implicit satisfaction measurement
   - Improvement suggestion tracking

### Evaluation Implementation

```python
class AgentEvaluator:
    """Evaluates agent outputs for quality and improvement."""
    
    def __init__(self):
        self.metrics = {
            "validation_rate": [],
            "completeness_scores": [],
            "response_times": [],
            "token_usages": [],
        }
    
    async def evaluate_output(
        self, 
        output: Any, 
        expected_type: Type[BaseModel], 
        execution_time: float,
        token_usage: Dict[str, int]
    ) -> Dict[str, float]:
        """Evaluate a single agent output."""
        result = {
            "validation_success": True,
            "completeness_score": 0.0,
            "response_time": execution_time,
            "token_usage": sum(token_usage.values()),
        }
        
        # Validation check
        try:
            if not isinstance(output, expected_type):
                # Try to convert/validate
                output = expected_type.model_validate(output)
            result["validation_success"] = True
        except ValidationError:
            result["validation_success"] = False
        
        # Completeness check
        result["completeness_score"] = self._calculate_completeness(output)
        
        # Update metrics history
        self.metrics["validation_rate"].append(int(result["validation_success"]))
        self.metrics["completeness_scores"].append(result["completeness_score"])
        self.metrics["response_times"].append(result["response_time"])
        self.metrics["token_usages"].append(result["token_usage"])
        
        return result
    
    def get_aggregate_metrics(self) -> Dict[str, float]:
        """Get aggregate metrics across all evaluations."""
        return {
            "avg_validation_rate": self._safe_average(self.metrics["validation_rate"]),
            "avg_completeness": self._safe_average(self.metrics["completeness_scores"]),
            "avg_response_time": self._safe_average(self.metrics["response_times"]),
            "avg_token_usage": self._safe_average(self.metrics["token_usages"]),
        }
    
    def _calculate_completeness(self, output: Any) -> float:
        """Calculate completeness score for an output."""
        if not output:
            return 0.0
            
        # For Pydantic models, check field completion
        if isinstance(output, BaseModel):
            fields = output.model_fields
            if not fields:
                return 1.0  # Empty model is "complete"
                
            # Count non-empty fields
            non_empty = 0
            for field_name, field_value in output.model_dump().items():
                if field_value is not None and field_value != "" and field_value != [] and field_value != {}:
                    non_empty += 1
                    
            return non_empty / len(fields)
            
        # For dictionaries
        if isinstance(output, dict):
            if not output:
                return 0.0
            # Count non-empty values
            non_empty = sum(1 for v in output.values() if v is not None and v != "" and v != [] and v != {})
            return non_empty / len(output)
            
        # For strings, check if non-empty
        if isinstance(output, str):
            return 1.0 if output.strip() else 0.0
            
        # Default for other types
        return 1.0 if output else 0.0
    
    def _safe_average(self, values: List) -> float:
        """Calculate average safely."""
        return sum(values) / len(values) if values else 0.0
        
    async def peer_evaluate(self, output: Any, evaluator_agent: Agent, ctx: RunContext) -> Dict[str, Any]:
        """Have another agent evaluate the output."""
        # Create evaluation prompt
        eval_prompt = f"""
Please evaluate the following output for quality, completeness, and correctness:

{json.dumps(output) if isinstance(output, (dict, list)) else str(output)}

Provide a score from 1-10 for each of these criteria:
1. Completeness: Does it include all necessary information?
2. Correctness: Is the information accurate and valid?
3. Clarity: Is it well-organized and easy to understand?
4. Actionability: Can it be implemented without further clarification?

Also provide specific feedback on how it could be improved.
"""
        
        # Run evaluator agent
        try:
            result = await evaluator_agent.run(eval_prompt, ctx)
            return result.data if hasattr(result, "data") else result
        except Exception as e:
            logger.error(f"Error in peer evaluation: {e}")
            return {
                "error": str(e),
                "scores": {
                    "completeness": 0,
                    "correctness": 0,
                    "clarity": 0,
                    "actionability": 0
                }
            }
```

## Context Window Management

### Memory Management

1. **Context Prioritization**
   - Identify and prioritize critical information
   - Summarize historical interactions
   - Prune less relevant details

2. **Document Chunking**
   - Split large documents into manageable chunks
   - Maintain cross-references between chunks
   - Reconstruct document semantic structure

3. **Compression Techniques**
   - Implement memory compression
   - Use summarization for long contexts
   - Create semantic embeddings for retrieval

### Token Budget Management

1. **Token Estimation**
   - Implement token counting for inputs
   - Create token budgets for different operations
   - Track token usage across interactions

2. **Optimization Strategies**
   - Truncate least relevant content
   - Replace content with references
   - Implement tiered context (detailed recent, summarized older)

3. **Token Saving Techniques**
   - Use shorthand for repeated references
   - Eliminate redundant information
   - Structure inputs for token efficiency

### Memory Implementation

```python
class ContextWindowManager:
    """Manages context window for efficient token usage."""
    
    def __init__(
        self, 
        max_tokens: int = 4000,
        reserve_tokens: int = 1000,
        prioritize_recent: bool = True
    ):
        self.max_tokens = max_tokens
        self.reserve_tokens = reserve_tokens
        self.prioritize_recent = prioritize_recent
        self.tokenizer = Tokenizer()  # Simplified; use actual tokenization
        
    def optimize_context(
        self,
        messages: List[Dict[str, str]],
        system_prompt: str,
        current_input: str
    ) -> List[Dict[str, str]]:
        """Optimize message history to fit context window."""
        # Calculate token budget
        system_tokens = self.tokenizer.count_tokens(system_prompt)
        input_tokens = self.tokenizer.count_tokens(current_input)
        available_tokens = self.max_tokens - system_tokens - input_tokens - self.reserve_tokens
        
        if available_tokens <= 0:
            logger.warning("No tokens available for context. Using empty context.")
            return []
            
        # Count tokens in each message
        message_tokens = [
            (i, msg, self.tokenizer.count_tokens(msg["content"]))
            for i, msg in enumerate(messages)
        ]
        
        # If messages fit within budget, return them all
        total_message_tokens = sum(tokens for _, _, tokens in message_tokens)
        if total_message_tokens <= available_tokens:
            return messages
            
        # Need to optimize context
        optimized_messages = []
        budget_used = 0
        
        if self.prioritize_recent:
            # Prioritize recent messages and add from newest to oldest until budget is exhausted
            for i, msg, tokens in reversed(message_tokens):
                if budget_used + tokens <= available_tokens:
                    optimized_messages.insert(0, msg)  # Insert at beginning to maintain order
                    budget_used += tokens
                else:
                    # Try to add a summarized version if it's a longer message
                    if tokens > 100:  # Only summarize substantial messages
                        summary = self._summarize_message(msg["content"])
                        summary_tokens = self.tokenizer.count_tokens(summary)
                        if budget_used + summary_tokens <= available_tokens:
                            summarized_msg = msg.copy()
                            summarized_msg["content"] = f"[SUMMARIZED] {summary}"
                            optimized_messages.insert(0, summarized_msg)
                            budget_used += summary_tokens
        else:
            # Prioritize by keeping message pairs (user+assistant) and important messages
            # More complex logic would go here, considering message importance
            # For simplicity, we'll just keep user-assistant pairs starting from most recent
            
            # Group into pairs (user followed by assistant)
            pairs = []
            current_pair = []
            
            for i, msg, tokens in reversed(message_tokens):
                current_pair.append((i, msg, tokens))
                if len(current_pair) == 2:
                    pairs.append(current_pair)
                    current_pair = []
            
            # Add any remaining message
            if current_pair:
                pairs.append(current_pair)
            
            # Add pairs from most recent until budget is exhausted
            for pair in pairs:
                pair_tokens = sum(tokens for _, _, tokens in pair)
                if budget_used + pair_tokens <= available_tokens:
                    # Add pair messages in original order
                    pair_msgs = sorted([(i, msg) for i, msg, _ in pair], key=lambda x: x[0])
                    for _, msg in pair_msgs:
                        optimized_messages.append(msg)
                    budget_used += pair_tokens
        
        return optimized_messages
    
    def chunk_document(self, document: str, max_chunk_tokens: int = 1000) -> List[str]:
        """Split document into manageable chunks."""
        chunks = []
        current_chunk = []
        current_size = 0
        
        # Split document into paragraphs
        paragraphs = document.split("\n\n")
        
        for paragraph in paragraphs:
            para_tokens = self.tokenizer.count_tokens(paragraph)
            
            # If single paragraph exceeds max chunk size, need to split further
            if para_tokens > max_chunk_tokens:
                # Further split into sentences
                sentences = re.split(r'(?<=[.!?])\s+', paragraph)
                for sentence in sentences:
                    sentence_tokens = self.tokenizer.count_tokens(sentence)
                    
                    # Handle very long sentences (split by character count as last resort)
                    if sentence_tokens > max_chunk_tokens:
                        # Split into smaller parts (simplified)
                        parts = [sentence[i:i+100] for i in range(0, len(sentence), 100)]
                        for part in parts:
                            part_tokens = self.tokenizer.count_tokens(part)
                            if current_size + part_tokens > max_chunk_tokens:
                                chunks.append("\n\n".join(current_chunk))
                                current_chunk = [part]
                                current_size = part_tokens
                            else:
                                current_chunk.append(part)
                                current_size += part_tokens
                    else:
                        # Handle normal sentences
                        if current_size + sentence_tokens > max_chunk_tokens:
                            chunks.append("\n\n".join(current_chunk))
                            current_chunk = [sentence]
                            current_size = sentence_tokens
                        else:
                            current_chunk.append(sentence)
                            current_size += sentence_tokens
            else:
                # Handle normal paragraphs
                if current_size + para_tokens > max_chunk_tokens:
                    chunks.append("\n\n".join(current_chunk))
                    current_chunk = [paragraph]
                    current_size = para_tokens
                else:
                    current_chunk.append(paragraph)
                    current_size += para_tokens
        
        # Add the last chunk if not empty
        if current_chunk:
            chunks.append("\n\n".join(current_chunk))
        
        return chunks
    
    def _summarize_message(self, message: str) -> str:
        """Create a shorter summary of a message."""
        # For a production system, this would use an LLM call
        # Here we use a simple heuristic for demonstration
        if len(message) <= 100:
            return message
            
        # Simple summarization: first sentence + length indicator
        first_sentence_match = re.match(r'^(.*?[.!?])\s', message)
        if first_sentence_match:
            first_sentence = first_sentence_match.group(1)
            return f"{first_sentence} [... {len(message) - len(first_sentence)} more chars]"
        
        # If no sentence break, take first 100 chars
        return message[:100] + f"... [{len(message) - 100} more chars]"
```

class Tokenizer:
    """Simplified tokenizer for demonstration."""
    
    def count_tokens(self, text: str) -> int:
        """Estimate token count for text."""
        # Very rough approximation: 4 chars per token
        return len(text) // 4 + 1
