# Dependency Visualization Guidelines

*This document outlines how dependencies between stories and components should be visualized and managed throughout the project.*

## Dependency Types

### Story Dependencies
- **Sequential**: One story must be completed before another can begin
- **Functional**: One story's functionality depends on another
- **Data**: One story relies on data structures created by another
- **Interface**: One story interacts with interfaces defined by another
- **Shared Resource**: Multiple stories affect the same resource

### Component Dependencies
- **Service Dependencies**: Services that components rely on
- **Library Dependencies**: External or internal code libraries
- **Configuration Dependencies**: Required configuration or environment settings
- **Infrastructure Dependencies**: Required infrastructure components

## Visualization Formats

### Dependency Graph
- Create a directed graph showing dependencies between stories
- Nodes represent stories
- Directed edges represent dependencies
- Label edges with dependency types
- Use color coding to indicate Epic groupings
- Include estimated complexity on each node

Example Mermaid syntax:
```
graph TD
    A[Story A-1] --> B[Story B-1]
    A --> C[Story A-2]
    B --> D[Story B-2]
    C --> D
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style C fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#bbf,stroke:#333,stroke-width:2px
    style D fill:#bbf,stroke:#333,stroke-width:2px
```

### Dependency Matrix
- Create a matrix where rows and columns are stories
- Mark cells where row story depends on column story
- Use different symbols or colors for different dependency types

Example:
```
          | Story A-1 | Story A-2 | Story B-1 | Story B-2 |
----------|-----------|-----------|-----------|-----------|
Story A-1 |     -     |     -     |     -     |     -     |
Story A-2 |     S     |     -     |     -     |     -     |
Story B-1 |     F     |     -     |     -     |     -     |
Story B-2 |     -     |     D     |     S     |     -     |
```
Legend: S = Sequential, F = Functional, D = Data, I = Interface

### Timeline Visualization
- Show stories on a timeline based on dependencies
- Highlight critical path
- Show parallel implementation opportunities
- Indicate complexity/effort by node size

## Dependency Management Practices

### Dependency Documentation
- Each dependency must be documented in both the dependent and dependency story
- Include the nature of the dependency and specific integration points
- Include references to specific file locations or interfaces

### Circular Dependency Prevention
- Actively identify and prevent circular dependencies
- If detected, refactor stories to eliminate circular relationships
- Document any intentional circular dependencies with justification

### Dependency Minimization
- Design stories to minimize dependencies when possible
- Consider creating interface/contract stories that other stories depend on
- Group highly interdependent functionality into the same story when appropriate

## Integration with Project Documentation

### Dependency Section in Story Template
Each story document should include a structured dependency section:
```
## Dependencies
- **Story Dependencies**:
  - [STORY-ID]: [Dependency Type] - [Specific Integration Point]
  - [STORY-ID]: [Dependency Type] - [Specific Integration Point]
- **Component Dependencies**:
  - [COMPONENT]: [Version] - [Dependency Type]
```

### Master Dependency Diagram
- A master dependency diagram should be maintained showing all stories
- The diagram should be updated whenever new stories are added
- The diagram should be available in both visual and machine-readable formats

## Tools for Dependency Visualization

### Recommended Tools
- **Mermaid**: For embedding diagrams in markdown
- **D3.js**: For interactive web-based visualizations
- **Graphviz**: For complex dependency graphs
- **Dependency matrices**: For comprehensive relationship mapping

### Integration with CLI
The CLI tool should be able to:
1. Generate dependency visualizations on demand
2. Detect circular or problematic dependencies
3. Show the impact of modifying a story on its dependents
4. Highlight the next implementable stories (no unmet dependencies)
