# ProjectPlanner - CLI Interface Design

This document specifies the command-line interface design for ProjectPlanner, detailing commands, interaction patterns, and user experience considerations.

## Interface Philosophy

The ProjectPlanner CLI follows these design principles:

1. **Progressive Disclosure**: Present simple commands first, reveal complexity as needed
2. **Consistent Patterns**: Use consistent naming and parameter patterns across commands
3. **Rich Feedback**: Provide clear, visually appealing feedback for all operations
4. **Graceful Errors**: Handle errors elegantly with actionable suggestions
5. **Intelligent Defaults**: Minimize required parameters with sensible defaults
6. **Accessibility**: Support different working styles and accessibility needs

## Core Commands

### Command Structure

All commands follow this general structure:

```
projectplanner [command] [subcommand] [options] [arguments]
```

### Command Categories

1. **Project Management**: Create and manage projects
2. **Agent Interaction**: Work directly with specialized agents
3. **Documentation**: Generate and export documentation
4. **System**: Configure and manage the ProjectPlanner system

### Global Options

These options are available in all commands:

| Option | Description |
|--------|-------------|
| `--help`, `-h` | Show help for a command |
| `--version`, `-v` | Show version information |
| `--config FILE` | Use specific configuration file |
| `--quiet`, `-q` | Suppress informational output |
| `--verbose` | Increase output verbosity |
| `--color [auto\|always\|never]` | Control color output |
| `--theme THEME` | Specify UI theme to use |
| `--output-format [text\|json\|yaml]` | Specify output format |

## Project Commands

### `new`

Creates a new project with initial setup.

```
projectplanner new PROJECT_NAME [options]
```

#### Options

| Option | Description |
|--------|-------------|
| `--template TEMPLATE` | Use specific project template (default: standard) |
| `--description TEXT` | Short project description |
| `--model MODEL` | AI model to use (default: gpt-4o) |
| `--no-interactive` | Skip interactive prompting |
| `--directory DIR` | Specify project directory |

#### Examples

```bash
# Create a new project with interactive prompting
projectplanner new inventory-system

# Create new project with specific template and model
projectplanner new inventory-system --template web-app --model claude-3-5-sonnet
```

### `continue`

Continue working on an existing project.

```
projectplanner continue PROJECT_PATH [options]
```

#### Options

| Option | Description |
|--------|-------------|
| `--agent AGENT` | Start with specific agent (default: next in workflow) |
| `--from-phase PHASE` | Start from specific phase |
| `--model MODEL` | Override AI model |
| `--no-interactive` | Run non-interactively |

#### Examples

```bash
# Continue working on a project with the next agent
projectplanner continue ./inventory-system

# Continue with a specific agent
projectplanner continue ./inventory-system --agent architect
```

### `export`

Export project documentation.

```
projectplanner export PROJECT_PATH [options]
```

#### Options

| Option | Description |
|--------|-------------|
| `--format [markdown\|html\|pdf]` | Output format (default: markdown) |
| `--output DIR` | Output directory (default: PROJECT_PATH/exports) |
| `--include-drafts` | Include draft documents |
| `--single-file` | Combine all docs into single file |
| `--developer-handoff STORY_ID` | Create developer handoff package for specific story |

#### Examples

```bash
# Export all project documentation as markdown
projectplanner export ./inventory-system

# Export specific story as developer handoff package
projectplanner export ./inventory-system --developer-handoff STORY-001-002
```

## Agent Commands

### `agent`

Work directly with a specific agent.

```
projectplanner agent AGENT_NAME PROJECT_PATH [options]
```

#### Options

| Option | Description |
|--------|-------------|
| `--prompt TEXT` | Initial prompt for the agent |
| `--interactive` | Start interactive session |
| `--input-file FILE` | Read input from file |
| `--output-file FILE` | Write output to file |
| `--model MODEL` | Override AI model |

#### Examples

```bash
# Start interactive session with business analyst
projectplanner agent business-analyst ./inventory-system --interactive

# Run architect with specific prompt
projectplanner agent architect ./inventory-system --prompt "Design a microservice architecture"
```

### `brainstorm`

Start brainstorming session with business analyst.

```
projectplanner brainstorm PROJECT_PATH [options]
```

#### Options

| Option | Description |
|--------|-------------|
| `--topic TEXT` | Specific topic to brainstorm |
| `--duration MINUTES` | Suggested session duration |
| `--focus [problem\|solution\|features]` | Session focus |
| `--save-transcript` | Save conversation transcript |

#### Examples

```bash
# Start general brainstorming session
projectplanner brainstorm ./inventory-system

# Focused feature brainstorming session
projectplanner brainstorm ./inventory-system --focus features --topic "inventory alerts"
```

### `decide`

Present architecture/design options and facilitate decision.

```
projectplanner decide PROJECT_PATH DECISION_TYPE [options]
```

#### Options

| Option | Description |
|--------|-------------|
| `--options FILE` | Load options from file |
| `--criteria FILE` | Load evaluation criteria from file |
| `--interactive` | Interactive decision process |
| `--export-decision` | Export decision with rationale |

#### Examples

```bash
# Interactively decide on architecture
projectplanner decide ./inventory-system architecture --interactive

# Decide based on predefined options and criteria
projectplanner decide ./inventory-system database --options db-options.yaml --criteria db-criteria.yaml
```

## System Commands

### `config`

Configure ProjectPlanner settings.

```
projectplanner config [SETTING] [VALUE] [options]
```

#### Options

| Option | Description |
|--------|-------------|
| `--list` | List all configuration settings |
| `--reset` | Reset to default settings |
| `--file FILE` | Specify config file |
| `--global` | Apply to global configuration |
| `--project PROJECT_PATH` | Apply to specific project |

#### Examples

```bash
# Set global API key
projectplanner config api_key.openai sk-123456 --global

# List project-specific settings
projectplanner config --list --project ./inventory-system
```

### `theme`

Manage UI themes.

```
projectplanner theme [SUBCOMMAND] [options]
```

#### Subcommands

| Subcommand | Description |
|------------|-------------|
| `list` | List available themes |
| `show THEME` | Show theme details |
| `set THEME` | Set active theme |
| `create` | Create new theme |
| `import FILE` | Import theme from file |
| `export THEME FILE` | Export theme to file |

#### Examples

```bash
# List available themes
projectplanner theme list

# Set active theme
projectplanner theme set cyberpunk

# Create new theme interactively
projectplanner theme create
```

### `plugins`

Manage plugins.

```
projectplanner plugins [SUBCOMMAND] [options]
```

#### Subcommands

| Subcommand | Description |
|------------|-------------|
| `list` | List installed plugins |
| `install PLUGIN` | Install plugin |
| `remove PLUGIN` | Remove plugin |
| `enable PLUGIN` | Enable plugin |
| `disable PLUGIN` | Disable plugin |
| `info PLUGIN` | Show plugin information |

#### Examples

```bash
# List installed plugins
projectplanner plugins list

# Install plugin
projectplanner plugins install github-integration
```

## Interactive Mode

### Chat Interface

The chat interface uses Rich for formatted display:

```
┌────────────────────────────────────────────────────────────────────┐
│ Project: Inventory System                                           │
└────────────────────────────────────────────────────────────────────┘

[Business Analyst]: Welcome to ProjectPlanner! I'll help you transform 
your idea into a clear project vision. What problem are you trying to 
solve?

You: I need a system to track inventory across multiple warehouses

[Business Analyst]: That sounds like an interesting challenge. Let me 
ask a few questions to understand better:

1. What kind of items will you be tracking?
2. How many warehouses do you need to manage?
3. Are there any specific features you need, like barcode scanning?

You: 
```

### Decision Interface

The decision interface presents options with comparisons:

```
┌─ Architecture Decision ──────────────────────────────────────────────┐
│                                                                      │
│ Based on your requirements, here are three architecture options:     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌─ Comparison ───────────────────────────────────────────────────────┐
│ Criteria      │ Microservices │ Monolithic   │ Serverless          │
│───────────────┼───────────────┼──────────────┼─────────────────────│
│ Scalability   │ High ✓        │ Medium       │ High ✓              │
│ Initial Cost  │ High          │ Low ✓        │ Low ✓               │
│ Maintenance   │ Medium        │ Low ✓        │ Low ✓               │
│ Deployment    │ Complex       │ Simple ✓     │ Medium              │
│ Performance   │ High ✓        │ High ✓       │ Variable            │
│ Flexibility   │ High ✓        │ Medium       │ Medium              │
└───────────────┴───────────────┴──────────────┴─────────────────────┘

┌─ Option 1: Microservices Architecture ───────────────────────────────┐
│                                                                      │
│ A distributed architecture with separate services for inventory      │
│ management, user authentication, reporting, and notifications.       │
│                                                                      │
│ Tech Stack: Python FastAPI, PostgreSQL, Redis, React, Docker, K8s    │
│                                                                      │
│ Pros:                                                                │
│ • Highly scalable                                                    │
│ • Independent deployment                                             │
│ • Technology flexibility                                             │
│                                                                      │
│ Cons:                                                                │
│ • Deployment complexity                                              │
│ • Higher initial setup cost                                          │
│ • Network overhead                                                   │
└──────────────────────────────────────────────────────────────────────┘

[Architect]: Which architecture option would you prefer for your 
inventory management system?

1. Microservices Architecture
2. Monolithic Architecture
3. Serverless Architecture
4. Tell me more about option...

Choose an option (1-4): 
```

### Progress Display

The progress display shows workflow status:

```
┌─ Project Status ───────────────────────────────────────────────────┐
│ Component            │ Status       │ Progress                     │
│─────────────────────┼──────────────┼─────────────────────────────│
│ Project Vision      │ Complete     │ 100%                         │
│ Requirements        │ Complete     │ 100%                         │
│ Architecture        │ In Progress  │ 60%                          │
│ Technical Standards │ In Progress  │ 40%                          │
│ User Stories        │ Not Started  │ 0%                           │
│ Implementation Plan │ Not Started  │ 0%                           │
└─────────────────────┴──────────────┴─────────────────────────────┘

Agent Status:
• Business Analyst: Complete
• Project Manager: Complete
• Architect: Active
• Technical Standards Coordinator: Active
• Scrum Master: Waiting

[••••••••••••••••······················] 60% Architecture Design
```

## Document Preview

The document preview uses syntax highlighting and formatting:

```
┌─ Technical Standards Document ─────────────────────────────────────┐
│                                                                    │
│ # Technical Standards                                              │
│                                                                    │
│ ## Coding Standards                                                │
│                                                                    │
│ ### Python                                                         │
│ - Use Python 3.10+ features                                        │
│ - Follow PEP 8 guidelines with Black formatter                     │
│ - Maximum line length: 88 characters                               │
│ - Use type annotations for all functions                           │
│                                                                    │
│ ### JavaScript                                                     │
│ - Use ES6+ features                                                │
│ - Follow Airbnb style guide                                        │
│ - Use TypeScript for complex components                            │
│                                                                    │
│ ## API Standards                                                   │
│                                                                    │
│ All APIs must follow RESTful design principles:                    │
│                                                                    │
│ | Method | Purpose                  |                              │
│ |--------|--------------------------|                              │
│ | GET    | Retrieve resources       |                              │
│ | POST   | Create new resources     |                              │
│ | PUT    | Update existing resources|                              │
│ | DELETE | Remove resources         |                              │
└────────────────────────────────────────────────────────────────────┘
```

## Story Breakdown View

The story breakdown view shows epics and stories:

```
┌─ EPIC-001: Inventory Tracking System ───────────────────────────────┐
│ Core functionality for tracking inventory items                     │
└──────────────────────────────────────────────────────────────────────┘

ID            | Story                 | Points | Status      
--------------|-----------------------|--------|-------------
STORY-001-001 | Item Registration     | 5      | Complete    
STORY-001-002 | Barcode Scanning      | 8      | In Progress 
STORY-001-003 | Stock Level Updates   | 3      | Not Started 
STORY-001-004 | Inventory Reports     | 5      | Not Started 

┌─ EPIC-002: User Management ─────────────────────────────────────────┐
│ User authentication and authorization system                        │
└──────────────────────────────────────────────────────────────────────┘

ID            | Story                 | Points | Status      
--------------|-----------------------|--------|-------------
STORY-002-001 | User Registration     | 3      | Not Started 
STORY-002-002 | Role-based Access     | 8      | Not Started 
STORY-002-003 | Password Reset        | 2      | Not Started 
```

## Error Handling

### Error Display

Errors are displayed with helpful context:

```
╭─ Error ─────────────────────────────────────────────────────────────╮
│ Unable to connect to the OpenAI API                                 │
│                                                                     │
│ Details: The API key provided is invalid or has been revoked.       │
╰─────────────────────────────────────────────────────────────────────╯

Suggestions:
• Check your API key in the configuration
• Verify your account has sufficient credits
• Ensure your internet connection is working

Run 'projectplanner config api_key.openai YOUR_KEY --global' to update your API key
```

### Warning Display

Warnings are displayed without interrupting flow:

```
⚠️  Warning: The selected model "gpt-4" has been deprecated. Using "gpt-4o" instead.
```

### Success Confirmation

Success messages provide clear confirmation:

```
✓ Project "inventory-system" created successfully!
  Next steps:
  • Run 'projectplanner continue inventory-system' to start requirements gathering
  • Run 'projectplanner agent business-analyst inventory-system --interactive' for brainstorming
```

## Theme System

### Theme Components

Themes customize these UI elements:

1. **Colors**
   - Primary: Main brand color
   - Secondary: Supporting color
   - Accent: Highlight color
   - Success/Warning/Error: Status colors
   - Background/Foreground: Base colors

2. **Styles**
   - Bold/Italic/Underline
   - Panel borders
   - Progress bars
   - Tables

3. **Syntax Highlighting**
   - Code blocks
   - Command syntax
   - Configuration files

### Theme Definition

Themes are defined in YAML or JSON:

```yaml
name: "Cyberpunk"
type: "dark"
description: "A neon-inspired cyberpunk theme with bright accents on dark background"

colors:
  primary: "#ff2a6d"      # Neon pink
  secondary: "#05d9e8"    # Cyan
  accent: "#d1f7ff"       # Light blue
  success: "#01c38d"      # Green
  warning: "#ff9f1c"      # Orange
  error: "#fb5607"        # Red-orange
  info: "#caf0f8"         # Light blue
  muted: "#6b7280"        # Gray
  background: "#0d1117"   # Very dark blue
  foreground: "#f0f6fc"   # Very light blue-gray
  highlight: "#7b2cbf"    # Purple

syntax:
  comment: "#6b7280"      # Gray
  keyword: "#ff2a6d"      # Neon pink
  string: "#01c38d"       # Green
  number: "#05d9e8"       # Cyan
  operator: "#ff9f1c"     # Orange
  function: "#d1f7ff"     # Light blue
  class_name: "#7b2cbf"   # Purple
  variable: "#f0f6fc"     # Light blue-gray
  background: "#0d1117"   # Very dark blue
  foreground: "#f0f6fc"   # Light blue-gray

ui:
  panel.border: "#ff2a6d"
  prompt: "bold #ff2a6d"
  link: "underline #05d9e8"
  heading: "bold #ff2a6d"
  subheading: "bold #05d9e8"
  sidebar.background: "#0d1117"
  sidebar.foreground: "#f0f6fc"
  sidebar.highlight: "#ff2a6d"
```

## Accessibility Considerations

### Color Contrast

All themes enforce WCAG AA-level contrast ratios between:
- Text and background
- UI elements and background
- Status indicators and background

### Alternative Interaction Modes

1. **Keyboard Navigation**
   - Full keyboard control for all interactions
   - Keyboard shortcuts for common actions
   - Focus indicators for keyboard users

2. **Screen Reader Support**
   - Semantic structure for screen readers
   - Alternative text for visual elements
   - ARIA attributes for dynamic content

3. **Cognitive Accessibility**
   - Clear, consistent command structure
   - Progressive disclosure of complexity
   - Helpful error messages and suggestions

## Installation and Initialization

### Installation Command

```bash
# Install from PyPI
pip install projectplanner

# Install development version
pip install git+https://github.com/projectplanner/projectplanner.git
```

### First-Run Experience

```
╭─ Welcome to ProjectPlanner ───────────────────────────────────────────╮
│                                                                       │
│ ProjectPlanner helps you transform ideas into comprehensive project   │
│ documentation through a team of specialized AI agents.                │
│                                                                       │
│ Let's set up your environment to get started.                         │
╰───────────────────────────────────────────────────────────────────────╯

API Configuration:
ProjectPlanner uses AI models for document generation. Which provider would you like to use?

1. OpenAI
2. Anthropic
3. Local Models (requires separate setup)

Enter your choice (1-3): 1

Enter your OpenAI API key: sk-********************************

✓ API configuration successful!

Default Settings:
• Projects directory: ~/projects
• Default model: gpt-4o
• Theme: dark

You can change these settings at any time with 'projectplanner config'

╭─ Next Steps ───────────────────────────────────────────────────────────╮
│                                                                        │
│ To create your first project:                                          │
│ projectplanner new my-project-name                                     │
│                                                                        │
│ For help with any command:                                             │
│ projectplanner --help                                                  │
│ projectplanner [command] --help                                        │
╰────────────────────────────────────────────────────────────────────────╯
```

## Command Implementation Strategy

Each command follows this structure:

1. **Command Definition**
   ```python
   @app.command()
   def new(
       project_name: str = typer.Argument(..., help="Name of the new project"),
       template: str = typer.Option("standard", "--template", "-t", help="Project template"),
       model: str = typer.Option(None, "--model", "-m", help="AI model to use"),
       description: str = typer.Option("", "--description", "-d", help="Project description"),
       no_interactive: bool = typer.Option(False, "--no-interactive", help="Skip interactive prompting"),
       directory: Path = typer.Option(None, "--directory", "-dir", help="Project directory")
   ):
       """Create a new project with initial setup."""
   ```

2. **Validation Logic**
   ```python
   # Validate project name
   if not is_valid_project_name(project_name):
       console.print(f"[error]Invalid project name: {project_name}[/error]")
       console.print("Project names must contain only letters, numbers, hyphens and underscores.")
       raise typer.Exit(1)
       
   # Resolve directory
   if directory is None:
       directory = Path(config.get("projects_dir", ".")) / project_name
   
   # Check if project already exists
   if directory.exists():
       console.print(f"[error]Project directory already exists: {directory}[/error]")
       raise typer.Exit(1)
   ```

3. **Execution Logic**
   ```python
   # Initialize core engine
   try:
       core_engine = CoreEngine(model=model)
   except Exception as e:
       console.print(f"[error]Failed to initialize engine: {e}[/error]")
       raise typer.Exit(1)
       
   # Create project
   with console.status("[primary]Creating project...[/primary]"):
       try:
           result = core_engine.initialize_project(
               project_name, 
               template=template,
               description=description,
               interactive=not no_interactive
           )
           
           # Initialize folder structure
           directory.mkdir(parents=True, exist_ok=True)
           # Set up project files
           
       except Exception as e:
           console.print(f"[error]Error creating project: {e}[/error]")
           raise typer.Exit(1)
   ```

4. **Result Presentation**
   ```python
   # Success message
   console.print(f"[success]✓ Project '{project_name}' created successfully![/success]")
   
   # Show project vision summary
   console.print("\n[bold]Project Vision:[/bold]")
   console.print(result.data.vision_summary)
   
   # Next steps
   console.print("\n[bold]Next steps:[/bold]")
   console.print(f"• Run '[cmd]projectplanner continue {project_name}[/cmd]' to begin requirements gathering")
   console.print(f"• Run '[cmd]projectplanner agent business-analyst {project_name} --interactive[/cmd]' for more brainstorming")
   ```

## Terminal Compatibility

The interface design considers these terminal environments:

1. **Full-Featured Terminals**
   - Modern terminal emulators
   - Full color support
   - Unicode support
   - Rich text formatting

2. **Limited Terminals**
   - Legacy terminal environments
   - Limited color support
   - ASCII-only mode
   - Simplified output formatting

3. **Screen Readers**
   - Non-visual terminal access
   - Clear, linear output
   - Semantic structure
   - Explicit state changes

## Implementation Notes

1. **Rich Integration**
   - Use Rich Console for all output
   - Configure theme integration
   - Create custom formatters for specialized content

2. **Typer Configuration**
   - Register completion for all commands
   - Configure help generation
   - Set up command groups

3. **Error Handling**
   - Use Typer's exception handling
   - Create consistent error formatting
   - Provide helpful recovery suggestions