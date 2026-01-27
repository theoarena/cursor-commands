---
name: agent-memory
description: "Use this skill when the user asks to save, remember, recall, or organize memories. Triggers on: 'remember this', 'save this', 'note this', 'what did we discuss about...', 'check your notes', 'clean up memories', 'make annotations' etc. (also in pt_BR). Also use proactively when discovering valuable findings worth preserving, including business rules, architectural decisions, and exceptions to project rules."
---

# Agent Memory

A persistent memory space for storing knowledge that survives across conversations.

**Location:** `.cursor/memories/`

On first use of this skill, ensure the base directory exists:

```bash
mkdir -p .cursor/memories
```

## Proactive Usage

Save memories when you discover something worth preserving:
- Research findings that took effort to uncover
- Non-obvious patterns or gotchas in the codebase
- Solutions to tricky problems
- Architectural decisions and their rationale
- In-progress work that may be resumed later
- **Project knowledge**: Business rules, architectural details, and exceptions to project rules (see Project Knowledge section below)

Check memories when starting related work:
- Before investigating a problem area
- When working on a feature you've touched before
- When resuming work after a conversation break
- When analyzing or implementing features that may have business rules or architectural constraints

Organize memories when needed:
- Consolidate scattered memories on the same topic
- Remove outdated or superseded information
- Update status field when work completes, gets blocked, or is abandoned

## Folder Structure

When possible, organize memories into category folders. No predefined structure - create categories that make sense for the content.

Guidelines:
- Use kebab-case for folder and file names
- Consolidate or reorganize as the knowledge base evolves
- **Project knowledge** must be stored in `.cursor/memories/project-knowledge/` (see Project Knowledge section)

Example:
```text
.cursor/
└── memories/
    ├── project-knowledge/
    │   ├── business-rules/
    │   │   ├── diagnostic-scoring.md
    │   │   └── user-permissions.md
    │   ├── architecture/
    │   │   ├── diagnostic-flow.md
    │   │   └── dimension-calculation.md
    │   └── exceptions/
    │       └── custom-form-validation.md
    ├── file-processing/
    │   └── large-file-memory-issue.md
    ├── dependencies/
    │   └── iconv-esm-problem.md
    └── project-context/
        └── december-2025-work.md
```

This is just an example. Structure freely based on actual content.

## Frontmatter

All memories must include frontmatter with a `summary` field. The summary should be concise enough to determine whether to read the full content.

**Summary is the decision point**: Agents scan summaries via `rg "^summary:"` to decide which memories to read in full. Write summaries that contain enough context to make this decision - what the memory is about, the key problem or topic, and why it matters.

**Required:**
```yaml
---
summary: "1-2 line description of what this memory contains"
created: 2025-01-15  # YYYY-MM-DD format
---
```

**Optional:**
```yaml
---
summary: "Worker thread memory leak during large file processing - cause and solution"
created: 2025-01-15
updated: 2025-01-20
status: in-progress  # in-progress | resolved | blocked | abandoned
tags: [performance, worker, memory-leak]
related: [src/core/file/fileProcessor.ts]
---
```

## Search Workflow

Use summary-first approach to efficiently find relevant memories:

```bash
# 1. List categories
ls .cursor/memories/

# 2. View all summaries
rg "^summary:" .cursor/memories/ --no-ignore --hidden

# 3. Search summaries for keyword
rg "^summary:.*keyword" .cursor/memories/ --no-ignore --hidden -i

# 4. Search by tag
rg "^tags:.*keyword" .cursor/memories/ --no-ignore --hidden -i

# 5. Full-text search (when summary search isn't enough)
rg "keyword" .cursor/memories/ --no-ignore --hidden -i

# 6. Read specific memory file if relevant
```

**Note:** Memory files are gitignored, so use `--no-ignore` and `--hidden` flags with ripgrep.

## Operations

### Save

1. Determine appropriate category for the content
   - **Project knowledge** (business rules, architecture, exceptions) → `.cursor/memories/project-knowledge/{category}/`
   - Other technical findings → `.cursor/memories/{category}/`
2. Check if existing category fits, or create new one
3. Write file with required frontmatter (use `date +%Y-%m-%d` for current date)

**For project knowledge:**
```bash
mkdir -p .cursor/memories/project-knowledge/business-rules/
# Note: Check if file exists before writing to avoid accidental overwrites
cat > .cursor/memories/project-knowledge/business-rules/filename.md << 'EOF'
---
summary: "Brief description of this business rule"
created: 2025-01-27
category: business-rules
domain: diagnostics
tags: [scoring, calculation]
related: [app/Services/DiagnosticService.php]
---

# Title

Content here...
EOF
```

**For other memories:**
```bash
mkdir -p .cursor/memories/category-name/
# Note: Check if file exists before writing to avoid accidental overwrites
cat > .cursor/memories/category-name/filename.md << 'EOF'
---
summary: "Brief description of this memory"
created: 2025-01-15
---

# Title

Content here...
EOF
```

### Maintain

- **Update**: When information changes, update the content and add `updated` field to frontmatter
- **Delete**: Remove memories that are no longer relevant
  ```bash
  trash .cursor/memories/category-name/filename.md
  # Remove empty category folders
  rmdir .cursor/memories/category-name/ 2>/dev/null || true
  ```
- **Consolidate**: Merge related memories when they grow
- **Reorganize**: Move memories to better-fitting categories as the knowledge base evolves

## Guidelines

1. **Write for resumption**: Memories exist to resume work later. Capture all key points needed to continue without losing context - decisions made, reasons why, current state, and next steps.
2. **Write self-contained notes**: Include full context so the reader needs no prior knowledge to understand and act on the content
3. **Keep summaries decisive**: Reading the summary should tell you if you need the details
4. **Stay current**: Update or delete outdated information
5. **Be practical**: Save what's actually useful, not everything

## Content Reference

When writing detailed memories, consider including:
- **Context**: Goal, background, constraints
- **State**: What's done, in progress, or blocked
- **Details**: Key files, commands, code snippets
- **Next steps**: What to do next, open questions

Not all memories need all sections - use what's relevant.

## Project Knowledge

**Location:** `.cursor/memories/project-knowledge/`

This is a living knowledge base for project-specific information that provides valuable context for development and analysis. Store here:

### Business Rules (`project-knowledge/business-rules/`)

Document domain-specific rules and constraints:
- How calculations work (e.g., diagnostic scoring algorithms)
- Validation rules beyond code (e.g., "users can only have one active subscription")
- Workflow constraints (e.g., "diagnostics must be completed before results are shown")
- Domain logic that isn't obvious from code alone
- Edge cases and special handling

**When to save:**
- When implementing or discovering a business rule that isn't self-evident
- When explaining why something works a certain way
- When documenting domain constraints that affect multiple features

### Architecture (`project-knowledge/architecture/`)

Document architectural decisions and patterns:
- Data flow patterns (e.g., "Controller → Manager → Service → Model")
- Component interaction patterns
- State management approaches
- Integration patterns with external services
- Performance considerations and optimizations
- Design patterns used and why

**When to save:**
- When making architectural decisions that deviate from standard patterns
- When documenting complex flows that span multiple layers
- When explaining why a particular architecture was chosen
- When documenting patterns that should be followed consistently

### Exceptions (`project-knowledge/exceptions/`)

Document cases where project rules are intentionally broken:
- Features that don't follow the standard Controller → Manager → Service flow
- Components that bypass hooks and call services directly
- Validation that happens outside FormRequest classes
- Any deviation from established patterns with rationale

**When to save:**
- When implementing a feature that breaks a project rule
- When documenting why a rule exception was necessary
- When explaining workarounds or temporary solutions
- When documenting technical debt that needs future attention

### Structure

Organize project knowledge by domain or feature area:

```text
project-knowledge/
├── business-rules/
│   ├── diagnostics.md
│   ├── dimensions.md
│   └── questionnaires.md
├── architecture/
│   ├── diagnostic-flow.md
│   ├── form-handling.md
│   └── state-management.md
└── exceptions/
    ├── legacy-feature-x.md
    └── temporary-workaround-y.md
```

### Frontmatter for Project Knowledge

Use enhanced frontmatter for better organization:

```yaml
---
summary: "How diagnostic scoring is calculated across dimensions"
created: 2025-01-27
category: business-rules  # business-rules | architecture | exceptions
domain: diagnostics  # Optional: feature/domain area
tags: [scoring, calculation, dimensions]
related: [app/Services/DiagnosticService.php, app/Models/Diagnostic.php]
---
```

### Search Project Knowledge

```bash
# Search business rules
rg "^summary:" .cursor/memories/project-knowledge/business-rules/ --no-ignore --hidden

# Search architecture decisions
rg "^summary:" .cursor/memories/project-knowledge/architecture/ --no-ignore --hidden

# Search exceptions
rg "^summary:" .cursor/memories/project-knowledge/exceptions/ --no-ignore --hidden

# Search by domain
rg "^domain:" .cursor/memories/project-knowledge/ --no-ignore --hidden | grep "diagnostics"
```

### Guidelines for Project Knowledge

1. **Be specific**: Include concrete examples, code references, and file paths
2. **Explain rationale**: Document why decisions were made, not just what was done
3. **Keep current**: Update when rules change or exceptions are resolved
4. **Cross-reference**: Link related knowledge entries and code files
5. **Make it actionable**: Write so future developers can understand and apply the knowledge
