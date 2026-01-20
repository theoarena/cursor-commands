# Start Task

## Overview

Initiate the execution of a specific PHASE/TASK from the active plan based on user input.

## Steps

1. **Plan Synchronization**
    - Read the current active plan file.
    - Locate the specific PHASE/TASK requested.
    - Verify that immediately preceding dependencies are resolved.
2. **Guideline Alignment**
    - Review project guidelines (e.g., `.cursor/rules/dev/development.mdc`).
    - Ensure the technical approach aligns with established architecture and conventions.
3. **Execution**
    - Begin development strictly focused on the identified PHASE/TASK scope.
    - Implement logic incrementally.
    - Update the plan status to "In Progress" (if applicable/requested).

## Start Task Checklist

- [ ] Target PHASE/TASK identified in the plan
- [ ] Prerequisites verified
- [ ] Project guidelines and architecture reviewed
- [ ] Development initiated within scope
