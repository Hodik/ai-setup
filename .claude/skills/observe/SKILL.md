---
description: Explore and understand codebase architecture, patterns, and design decisions
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
---

Explore and understand the codebase, then optionally update documentation.

Focus area: $ARGUMENTS

## Role: Exploration and Understanding Only

**DO NOT**:

- Debug specific issues
- Create plans for new features
- Propose fixes or changes
- Generate implementation tasks

**DO**:

- Explore existing patterns and design decisions
- Map relationships between components
- Understand architectural choices
- Document data flows and dependencies
- Identify conventions and practices
- Note how different modules interact

## Workflow

### 1. Read Documentation First

Read documentation files directly (from `docs/` folder) to understand existing documented patterns before exploring code:

- `docs/system_patterns.md` - Architectural patterns and design decisions
- `docs/conventions.md` - Naming conventions, file structure, standard practices
- `docs/tech_context.md` - Technologies, constraints, dependencies
- `docs/formatting.md` - Code style and formatting rules

### 2. Explore the Codebase

Examine:

- File structure and organization
- Component relationships and dependencies
- Design patterns in use
- Naming conventions and coding standards
- Data flows and architectural decisions

### 3. Analyze and Synthesize

Identify:

- High-level architectural patterns
- Design principles evident in the code
- Trade-offs and rationale behind decisions
- Common abstractions and conventions

### 4. Prepare to Discuss

- Summarize findings in a clear, structured way
- Be ready to answer questions about what you observed
- Present trade-offs and rationale behind design choices
- Explain the "why" and "how" of the current implementation

## Output Format

Present your observations in this structured format:

### Architecture & Design

- High-level patterns used
- Design principles evident in the code

### Component Relationships

- How modules/classes interact
- Key dependencies and data flows

### Conventions & Patterns

- Coding patterns consistently used
- Naming conventions
- Common abstractions

### Notable Decisions

- Interesting architectural choices
- Trade-offs that were made
- Framework/library usage patterns

## Conditional: Update Documentation

If you discovered significant undocumented patterns or architectural decisions:

1. **Identify the right docs/ file** to update:
   - `docs/system_patterns.md` - For architectural patterns, design decisions
   - `docs/tech_context.md` - For technologies, dependencies, constraints
   - `docs/conventions.md` - For naming conventions, file structure patterns
   - `docs/formatting.md` - For formatting rules, language versions

2. **Update guidelines**:
   - Add to existing files when the update fits the file's context
   - Keep module/system structure - organize by components, not chronologically
   - Only add important changes - architectural decisions, patterns, constraints
   - Update existing sections rather than creating duplicate content
   - Keep docs concise and scannable
   - Document **why** decisions were made, not just what was done

3. **Do NOT document**:
   - Temporary fixes or workarounds
   - Implementation details obvious from the code
   - Chronological project history
   - Information already in other docs
