# JTBD & UX Elicitation Reference

Use this when you need deeper guidance for Phase 1–3 (analysis, task elicitation, business rule elicitation). Keep SKILL.md as the source of truth for the workflow.

---

## JTBD Quick Lens

- **Job**: “When I ___[situation]___, I want to ___[motivation]___, so I can ___[outcome]___.”
- **Forces**: Push (frustration), Pull (attraction), Anxiety (fear of change), Habit (current behavior).
- **Outcomes**: Functional (what gets done), Emotional (how they feel), Social (how they’re seen).
- **Stakeholders**: Job executor vs. decision maker vs. influencer; B2B vs. B2C.

Use this to structure analysis and to spot missing context (situation, motivation, outcome, forces).

---

## Task Elicitation (Phase 2)

**Goal**: Turn the job into concrete, ordered tasks.

**Prompts**:
- “Walk me through what happens from the moment the user decides to do this until they’re done.”
- “What does the user do vs. what does the system do?”
- “What has to be true before step X? What happens right after?”
- “How do they know each step is complete?”
- “Who can start this? Who can approve/finish it?”

**Watch for**: Hidden steps, parallel vs. sequential flows, handoffs between user and system.

---

## Business Rule Elicitation (Phase 3)

**Goal**: Extract rules that will govern implementation and tests.

**Prompts**:
- “What’s allowed? What’s forbidden?”
- “What happens if they leave X empty? If they enter invalid Y?”
- “Can the same thing happen twice? How do we handle duplicates?”
- “Who can do this? Who cannot? Does it depend on role, state, or time?”
- “Are there calculations or totals? How are they computed?”
- “Does this depend on other features or external systems? How?”

**Categories**:
- **Validation**: formats, ranges, required vs. optional.
- **Permissions**: who, when, on what.
- **State**: lifecycle, transitions, final states.
- **Edge cases**: empty, duplicate, concurrent, conflicting.
- **Derived data**: formulas, aggregations, defaults.

---

## Red Flags

- Moving to Phase 4 with vague tasks (“user does stuff”).
- Compiling rules without explicit edge cases and “what if” answers.
- Skipping user approval (Phase 4) and going straight to the dev plan.
- Writing the dev plan with Step 1 ≠ Pest tests for business rules.
