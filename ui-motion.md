# UI Motion Polish

## Overview
This command focuses on elevating User Interface (UI) quality through animations, transitions, and visual effects. The goal is to create a fluid, harmonious, and performant ("butter smooth") experience, avoiding robotic and linear movements.

**Target Stack:** Tailwind CSS and Pure CSS (Sass).
**Priority:** Elegant simplicity, natural acceleration curves (cubic-bezier), and performance.

## Steps

1.  **Scope and Intent Analysis**
    - Analyze the user input to identify if a specific scope was requested (e.g., "animate modal entry", "page transition").
    - **Decision Rule:** If the user does **NOT** provide an explicit scope, assume the focus is on **Micro-interactions** (hover effects, active states, focus rings, loading spinners, button feedback).

2.  **Curves and Timing Definition (The "Feel")**
    - **Forbidden:** Avoid `linear` for visible movements (use only for continuous spinners or pure opacity if necessary).
    - **Standard:** Use natural curves.
        - For Tailwind, prefer `ease-out`, `ease-in-out`, or extend the theme with `cubic-bezier(0.4, 0, 0.2, 1)`.
        - For Sass/CSS, use variables for easings: `transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);`.
    - **Duration:** Keep micro-interactions fast (150ms - 300ms). Larger transitions can go up to 400ms-500ms but must remain snappy.

3.  **Technical Implementation (Performance First)**
    - **Properties:** Preferably animate `transform` (translate, scale, rotate) and `opacity`. Avoid animating `width`, `height`, `margin`, or `top/left` to prevent *layout thrashing*.
    - **Tailwind:** Use composite utility classes (e.g., `transition-all duration-300 ease-in-out hover:scale-105 active:scale-95`).
    - **Sass/CSS:** Ensure complex transitions use `will-change` sparingly if needed to promote GPU layers.

4.  **Accessibility and Robustness**
    - Ensure the application maintains core functionality. Animation should be "progressive enhancement," not a blocker.
    - Respect `prefers-reduced-motion`. In Tailwind: `motion-reduce:transition-none` or `motion-reduce:transform-none`.

5.  **Visual Refinement**
    - Ensure animations are subtle. "Less is more."
    - Check for consistency with the rest of the existing design system.

## UI Motion Checklist
- [ ] Was the scope correctly identified (Micro vs Macro)?
- [ ] Does the animation use natural deceleration curves (e.g., cubic-bezier/ease-out) and avoid `linear`?
- [ ] Is performance optimized (focus on opacity/transform)?
- [ ] Does the code respect `prefers-reduced-motion`?
- [ ] Is the result visually simple and non-distracting?
- [ ] Does the code follow project conventions (Tailwind or Sass)?