---
name: jtbd-to-feature
description: Transforms JTBD (Jobs to be Done) sentences into system functionalities through structured discovery. Analyzes JTBD with UX/UX Research lens, elicits tasks and business rules via conversation, compiles rules for user approval, then produces a development plan with Pest tests first. Use when the user provides JTBD sentences to implement, wants to map JTBD to features, or asks to go from jobs-to-be-done to implementation.
---

# JTBD to Feature

Transforms JTBD sentences into implementable functionalities through analysis, conversation, and approved business rules. Follow the phases **in order**. Do not skip phases or move forward without user approval where required.

## Phase 1: Receive and Analyze

1. Receive one or more JTBD sentences from the user.
2. Analyze using JTBD, UX Design, and UX Research knowledge:
   - **Job**: What progress is the user trying to make? (functional, emotional, social)
   - **Context**: Situation, triggers, forces (push/pull/anxiety/habit)
   - **Outcomes**: Desired end state, success criteria
   - **Stakeholders**: Who is the “user” in the job? Who else is affected?
3. Summarize the analysis and **any ambiguities or assumptions** before moving on.

**Output**: A short, structured analysis (job, context, outcomes, open questions).

---

## Phase 2: Task Elicitation

1. Start a **conversation** about the JTBD sentences.
2. Ask questions to clarify and obtain more information about the **tasks** the JTBDs represent:
   - What concrete actions must the user (or system) perform?
   - In what order? What are the dependencies?
   - What does “done” look like for each task?
   - Who does what? (user vs. system)
3. Resolve doubts and fill gaps. Do **not** proceed until tasks are sufficiently clear.

**Output**: A clear list of tasks (who, what, when, done-when). No formal approval gate here—conversation continues until you and the user agree tasks are understood.

---

## Phase 3: Business Rule Elicitation

1. With tasks in hand, run a **second conversation** focused on **business rules**.
2. Ask questions to gather enough information to define:
   - Validation rules (inputs, formats, boundaries)
   - Permissions and constraints (who can do what, when)
   - State transitions (e.g. draft → submitted → approved)
   - Edge cases (empty data, duplicates, conflicts)
   - Aggregations, calculations, or derived values
   - Integration points (other features, external systems)
3. Use the same conversational style: clarify, challenge assumptions, surface edge cases.

**Output**: A **compiled list of business rules** (see format below).

---

## Phase 4: Compile and Approve

1. **Compile** the elicited information into **business rules**.
2. Use a consistent format. Example structure:
   - **Rule id** (optional): short slug
   - **Description**: what the rule enforces
   - **When it applies**: context, triggers
   - **Inputs / Outputs**: relevant data
   - **Edge cases / Exceptions**: if any
3. Present the compiled rules to the user and **ask explicitly**: “Are these business rules approved?”
4. **If approved** → proceed to Phase 5.
5. **If not approved** → iterate: discuss, refine, add/remove rules, and ask again until the user approves.

**Output**: An approved set of business rules. No development plan until approval.

---

## Phase 5: Development Plan

1. With **approved business rules**, elaborate a **development plan** for the functionality.
2. **First step of the plan must be**: Create automated **Pest** tests that exercise the mapped business rules.
   - Use the project’s Pest conventions and the **pest-tests** cursor command where applicable.
   - Tests should cover happy path, failure paths, and edge cases implied by the rules.
   - Prefer feature tests for HTTP/database flows; unit tests for isolated logic.
3. Subsequent steps: implement backend (Controller → Manager → Service → Model), frontend (Component → Hook → Service), validation, etc., following existing project architecture (e.g. Laravel Boost, laravel-react-inertia-dev skill).

**Output**: A numbered plan. Step 1 = Pest tests for business rules; remaining steps = implementation.

---

## Checklist (Do Not Skip)

- [ ] Phase 1: JTBD analyzed; analysis summarized.
- [ ] Phase 2: Tasks elicited via conversation; tasks clear.
- [ ] Phase 3: Business rules elicited via conversation.
- [ ] Phase 4: Rules compiled; user explicitly approved.
- [ ] Phase 5: Plan created; **Step 1 = Pest tests for business rules**.

---

## Business Rules Format (Reference)

When compiling rules in Phase 4, you can align with project-style docs. Example (see `.cursor/memories/project-knowledge/business-rules/`):

```markdown
## [Rule name]

### When it applies
- Context, triggers.

### Description
- What is enforced.

### Inputs / Outputs
- Relevant data, valid ranges.

### Edge cases / Exceptions
- Empty, duplicates, conflicts, etc.
```

---

## Additional Resources

- For JTBD and UX elicitation techniques (interviewing, forces, outcome-driven prompts), see [reference.md](reference.md).
