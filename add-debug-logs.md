# Add Debug Logs

## Overview
Strategically inserts detailed debug log statements into the codebase to visualize execution flow and state. It dynamically adapts to the project's logging standards and strictly adheres to the requested scope and formatting.

## Steps
1. **Context & Tool Analysis**
   - **Scope Detection:** If a specific block/function is selected, restrict actions to that scope. If no selection is made, apply to the logical flow of the currently active task/function.
   - **Log System Identification:** Analyze existing imports, configuration files, and neighboring code to determine the idiomatic logging method (e.g., `console.log`, `logger.info`, `Log::debug`, `print()`). **Do not** introduce new libraries.

2. **Flow Mapping**
   - Identify critical "checkpoints" in the execution flow:
     - **Entry:** Function start (log input arguments).
     - **Branching:** Inside `if/else`, `switch`, and `loops`.
     - **State Changes:** Before/After complex mutations or API calls.
     - **Exit/Error:** Return statements and Catch blocks.

3. **Log Injection**
   - Insert log statements at mapped checkpoints.
   - **Strict Formatting:** Every log message **MUST** start with the prefix `[DEBUG LOG]`.
   - **Structure:** `[DEBUG LOG] <Context/Function> | <Step Name> | Data: <Variables>`
   - Ensure variable values are readable (e.g., serialized objects).

## Debug Log Checklist
- [ ] Log messages use the project's native logging system (no new deps).
- [ ] Every log string starts with the exact prefix `[DEBUG LOG]`.
- [ ] Logs cover Entry, Exit, and Branching points to recreate the "story" of execution.
- [ ] Scope was strictly respected (selection vs. active flow).
