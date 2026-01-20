# Continue Task

## Overview

Resume execution of the currently active PHASE/TASK from the exact point of interruption until completion.

## Steps

1. **State Analysis**
    - Identify the active "In Progress" task in the plan.
    - Analyze the codebase to pinpoint exactly where development stopped.
    - Verify what has been implemented vs. what remains in the task scope.
2. **Resumption**
    - Resume coding immediately from the last completed logical step.
    - Maintain strict adherence to project guidelines (e.g., `.cursor/rules/dev/development.mdc`).
    - Avoid redundancy or overwriting valid existing code.
3. **Completion**
    - Continue implementation iteratively until the task requirements are fully met.
    - Update the plan status to "Completed" once finished.

## Continue Task Checklist

- [ ] Interruption point accurately identified
- [ ] Development resumed without redundancy
- [ ] Project guidelines strict adherence maintained
- [ ] Task scope fully implemented and verified
