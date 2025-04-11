# Project Documentation Output Structure with Pydantic AI

*This document defines the standardized folder structure and file organization for all documentation produced by ProjectPlanner, now enhanced with Pydantic AI validation and schema enforcement.*

## Root Folder Structure

```
ProjectName/
├── 1_Project_Overview/
├── 2_Requirements/
├── 3_Research/
├── 4_Architecture/
├── 5_Design/
├── 6_Technical_Standards/
├── 7_Task_List/
├── 8_Testing/
├── 9_DevOps/
├── 10_Implementation/
│   ├── Epics/
│   └── Stories/
└── .projectplanner/
    ├── schemas/
    ├── state/
    └── config.yaml
```

The new `.projectplanner` directory contains Pydantic schemas, state persistence for workflow tracking using Pydantic Graph, and configuration files.

## Document Validation with Pydantic Models

Each document category now has an associated Pydantic model that defines its schema. This ensures that all documents:

1. Contain the required fields
2. Maintain proper data types
3. Follow cross-document reference requirements
4. Pass custom validators for domain-specific rules

Example schema definitions:

```python
from datetime import datetime
from typing import List, Optional
from pydantic import BaseModel, Field, validator

class Stakeholder(BaseModel):
    name: str
    role: str
    influence: str
    interests: List[str]

class ProjectVision(BaseModel):
    title: str
    version: str
    date: datetime
    problem_statement: str
    proposed_solution: List[str]
    key_stakeholders: List[Stakeholder]
    success_criteria: List[str]
    
    @validator('problem_statement')
    def problem_statement_min_length(cls, v):
        """Ensure problem statement is substantial."""
        if len(v.split()) < 20:
            raise ValueError("Problem statement must be at least 20 words")
        return v

class FunctionalRequirement(BaseModel):
    id: str
    description: str
    priority: str
    stakeholders: List[str]
    acceptance_criteria: List[str]
    
    @validator('id')
    def validate_id_format(cls, v):
        """Ensure ID follows FR-### format."""
        if not v.startswith('FR-') or not v[3:].isdigit():
            raise ValueError("Functional requirement ID must follow FR-### format")
        return v
```

These schemas are used by both:
1. The Pydantic AI agents when generating documents (as result_type)
2. The Document Store when validating and saving documents

## Folder Contents with Enhanced Validation

### 1_Project_Overview/
```
1_Project_Overview/
├── Project_Vision.md           # High-level project vision from Business Analyst
├── Executive_Summary.md        # Executive summary of the project
├── Stakeholders.md             # Identified stakeholders and their interests
└── Glossary.md                 # Project-specific terminology and definitions
```

Each document is now linked to a Pydantic model for validation and a set of agents that can modify it:

```python
document_metadata = {
    "Project_Vision.md": {
        "schema": "ProjectVision",
        "agents": ["business_analyst"],
        "required": True,
        "dependencies": []
    },
    "Executive_Summary.md": {
        "schema": "ExecutiveSummary",
        "agents": ["business_analyst", "project_manager"],
        "required": True,
        "dependencies": ["Project_Vision.md"]
    },
    # Additional document metadata...
}
```

### 4_Architecture/
```
4_Architecture/
├── Architecture_Overview.md    # High-level architecture summary
├── Tech_Stack.md               # Selected technology stack with justifications
├── System_Architecture/        # Detailed system architecture documentation
│   ├── Component_Diagrams/
│   ├── Sequence_Diagrams/
│   └── Data_Flow_Diagrams/
├── Data_Architecture/          # Data models and database architecture
│   ├── Entity_Relationship_Diagrams/
│   ├── Database_Schemas/
│   └── Data_Dictionary.md
├── API_Specifications/         # API definitions and contracts
├── Integration_Points.md       # External system integration specifications
└── Security_Architecture.md    # Security design and implementation
```

The Architecture documents leverage Pydantic models extensively for validation of technical specifications, ensuring consistency across components:

```python
class Component(BaseModel):
    name: str
    description: str
    responsibilities: List[str]
    dependencies: List[str]
    technologies: List[str]
    interfaces: List[dict]

class TechStackItem(BaseModel):
    name: str
    category: str
    version: str
    justification: str
    alternatives_considered: List[str]
    decision_factors: List[str]

class SystemArchitecture(BaseModel):
    overview: str
    architectural_style: str
    components: List[Component]
    key_decisions: List[dict]
    constraints: List[str]
```

## Story Package Structure

Each story in the Stories folder is now generated using Pydantic AI agents and validated against Pydantic models:

```
Stories/EPIC-XXX/STORY-XXX-YYY/
├── STORY-XXX-YYY.md            # The comprehensive story document
├── Resources/                  # Story-specific resources
│   ├── Mockups/                # UI mockups relevant to this story
│   ├── Diagrams/               # Technical diagrams for this story
│   └── Sample_Data/            # Sample data for implementation and testing
├── Dependencies.md             # Specific dependencies for this story
└── Technical_Standards_Application.md  # How technical standards apply to this story
```

Each story is generated using a `StoryTemplate` Pydantic model:

```python
class StoryTemplate(BaseModel):
    story_id: str
    title: str
    context: dict
    dependencies: List[str]
    technical_specs: dict
    implementation_details: dict
    integration_points: dict
    acceptance_criteria: List[str]
    testing_guidelines: dict
    additional_resources: dict
```

This ensures that all stories follow a consistent format and have all required information for implementation.

## Cross-Reference System with Validation

To maintain traceability across documents, we've implemented a cross-reference system using Pydantic for validation:

```python
class DocumentReference(BaseModel):
    doc_id: str
    section: Optional[str] = None
    anchor: Optional[str] = None
    
    @validator('doc_id')
    def validate_doc_exists(cls, v):
        """Ensure referenced document exists."""
        if not document_store.document_exists(v):
            raise ValueError(f"Referenced document {v} does not exist")
        return v
        
class TraceabilityItem(BaseModel):
    source_doc: DocumentReference
    target_doc: DocumentReference
    relationship_type: str
    description: Optional[str] = None
    
class TraceabilityMatrix(BaseModel):
    items: List[TraceabilityItem]
    
    def get_dependencies(self, doc_id: str) -> List[DocumentReference]:
        """Get all documents that the specified document depends on."""
        return [item.target_doc for item in self.items 
                if item.source_doc.doc_id == doc_id]
                
    def get_dependents(self, doc_id: str) -> List[DocumentReference]:
        """Get all documents that depend on the specified document."""
        return [item.source_doc for item in self.items 
                if item.target_doc.doc_id == doc_id]
```

This powerful traceability system allows for:
1. Automated validation of cross-references
2. Impact analysis when making changes
3. Dependency tracking for document generation
4. Visualization of relationships between documents

## Document Generation Process

The document generation process has been enhanced with Pydantic AI:

```python
async def generate_document(doc_type: str, context: dict, deps: dict):
    """Generate a document using the appropriate agent."""
    # Get document metadata
    metadata = document_metadata.get(doc_type)
    if not metadata:
        raise ValueError(f"Unknown document type: {doc_type}")
        
    # Get the schema
    schema_class = get_schema_class(metadata["schema"])
    
    # Get the appropriate agent
    agent = agent_manager.get_agent(metadata["agents"][0])
    
    # Prepare the prompt
    prompt = f"Generate a {doc_type} document based on the following context:\n"
    prompt += json.dumps(context)
    
    # Run the agent with the appropriate schema as result_type
    result = await agent.run(prompt, deps=deps)
    
    # The result is already validated by Pydantic AI against the schema
    document_data = result.data
    
    # Generate the document using the template engine
    document_content = template_engine.render_template(
        doc_type, document_data.model_dump()
    )
    
    # Save the document
    document_store.save_document(
        Document(
            metadata=DocumentMetadata(
                id=doc_type,
                title=document_data.title if hasattr(document_data, "title") else doc_type,
                version="1.0",
                created_at=datetime.now(),
                updated_at=datetime.now(),
                created_by=metadata["agents"][0]
            ),
            content=document_content,
            schema_name=metadata["schema"]
        )
    )
    
    return document_content
```

## Additional Documentation for Developer Agent Handoff

When preparing documentation for a Developer Agent, we use Pydantic AI to generate a fully validated package:

```python
async def generate_implementation_package(story_id: str):
    """Generate an implementation package for a developer agent."""
    # Get the story
    story = document_store.get_document(story_id)
    
    # Get the technical standards
    tech_standards = document_store.get_document("Technical_Standards.md")
    
    # Create the implementation package structure
    package = ImplementationPackage(
        story=story,
        technical_standards=tech_standards,
        context=generate_context(story),
        resources=gather_resources(story),
        readme=generate_readme(story)
    )
    
    # Validate the package is complete and self-contained
    validator = ImplementationPackageValidator()
    validation_result = validator.validate(package)
    
    if not validation_result.is_valid:
        # Generate missing components
        await generate_missing_components(package, validation_result.missing_components)
        
    # Export the package
    return export_implementation_package(package)
```

## Story Readiness Checklist Integration

The Story Readiness Checklist is now implemented as a Pydantic validator for Story documents:

```python
class StoryReadinessValidator(BaseModel):
    """Validator for story readiness."""
    
    story: StoryTemplate
    
    def validate(self) -> dict:
        """Validate that the story is ready for implementation."""
        results = {
            "basics": self._validate_basics(),
            "context": self._validate_context(),
            "technical": self._validate_technical(),
            "integration": self._validate_integration(),
            "quality": self._validate_quality(),
            "standards": self._validate_standards(),
            "guidance": self._validate_guidance(),
            "completeness": self._validate_completeness()
        }
        
        return {
            "is_ready": all(results.values()),
            "results": results,
            "missing_items": self._get_missing_items(results)
        }
        
    def _validate_basics(self) -> bool:
        """Validate story basics."""
        # Story ID format validation
        id_valid = re.match(r"STORY-\d{3}-\d{3}", self.story.story_id)
        
        # Title validation
        title_valid = len(self.story.title) > 10
        
        # Size validation
        size_valid = len(self.story.implementation_details) > 0
        
        return id_valid and title_valid and size_valid
        
    # Additional validation methods for each section...
```

This validator is used by the Scrum Master agent to verify that each story is fully ready for implementation before it's finalized.

## Templates and Standards Integration

With Pydantic AI integration, the relationship between Story documents and Technical Standards is now more tightly coupled:

```python
class TechnicalStandardsApplication(BaseModel):
    """How technical standards apply to a specific story."""
    
    applicable_standards: List[dict]
    exceptions: List[dict]
    
    @validator('exceptions')
    def validate_exceptions(cls, v, values):
        """Validate that exceptions are properly documented."""
        if not v:
            return v
            
        for exception in v:
            if "justification" not in exception or len(exception["justification"]) < 20:
                raise ValueError(
                    f"Exception {exception.get('standard', 'unknown')} lacks proper justification"
                )
                
        return v
```

This ensures that when standards are referenced or exceptions are made, they are properly documented and justified.

## Implementation Considerations

The adoption of Pydantic AI has enhanced our documentation structure with:

1. **Strong Validation**: All documents are validated against Pydantic models
2. **Consistency**: Unified structure across all documents 
3. **Traceability**: Enhanced cross-referencing with validation
4. **Agent Integration**: Direct mapping between documents, schemas, and the agents that generate them
5. **Testing**: Ability to test document generation against schema requirements
6. **Quality Assurance**: Built-in validation for completeness and correctness

This structured approach ensures that all documentation produced by ProjectPlanner is consistent, complete, and ready for implementation by developer agents.
