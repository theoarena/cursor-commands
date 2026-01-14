# Quick Review

## Overview

Perform a fast but effective sanity check on recent changes. Ensure adherence to project guidelines, removing redundancies, and verifying that nothing was missed or added by mistake. 

## Steps

1. **Project Standards Check**
    - Verify strict adherence to project guidelines (e.g., `.cursor/rules/dev/development.mdc`).
    - Ensure file naming, folder structure, and formatting match the project's conventions.
2. **Cleanup & Redundancy**
    - Identify and remove unused imports, variables, or "dead code".
    - Check for accidental artifacts (console logs, temporary comments, AI-generated slop).
    - Ensure no code was added outside the requested scope.
3. **Logical Sanity Check**
    - Briefly verify that the changes actually solve the requested task.
    - Check for obvious omissions or edge cases that were forgotten.
    - Ensure code simplicity (no over-engineering).

## Quick Review Checklist

- [ ] Project guidelines and conventions respected
- [ ] No redundancies, unused code, or accidental artifacts found
- [ ] Scope respected (nothing added by mistake)
- [ ] Task requirements met (nothing forgotten)
- [ ] Code is clean and idiomatic
