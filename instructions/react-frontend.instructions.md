---
description: Reference React skill and design system skill before implementing frontend code.
applyTo: 'app/src/**/*.{ts,tsx}'
---

Before implementing or modifying React code:

1. **Load applicable skills:**
   - `react` — Architecture, components, state management, API layer, performance patterns
   - `design-system` — Colors, typography, spacing, MUI theming, component specs

2. **Load skill reference guides** for the specific task:
   - `custom-hooks.md` — When creating hooks or page logic
   - `component-patterns.md` — When creating components
   - `component-granularity.md` — When a component grows large (~200+ lines) or has repeated/conditional UI
   - `compound-components.md` — When building multi-part components (Card, Accordion, etc.)
   - `typescript-safety.md` — When defining props, event handlers, or request/response types
   - `project-structure.md` — When deciding where files or feature modules belong
   - `api-layer.md` — When adding API calls or queries
   - `state-management.md` — When managing complex state
   - `useeffect.md` — When using effects or subscriptions
   - `performance.md` — When optimizing (code splitting, memoization, re-renders)
   - `testing-strategy.md` — When writing or updating tests

3. **Follow conventions in** [app/AGENTS.md](../../../app/AGENTS.md)

4. **Audit the touched unit against the React skill.** When adding a feature
   or changing an existing component, hook, or function, review the *entire*
   unit you touched (not just the new lines) against the `react` skill and its
   reference guides. If you find meaningful drift from the conventions (e.g.
   oversized component, missing custom hook, prop drilling, inline styles,
   misplaced files):
   - Briefly summarize what drifts and why it matters.
   - **Ask the user whether they want a refactor** before doing one — do not
     refactor unprompted.
   - Keep the original change scoped; treat the refactor as a separate,
     opt-in step. Skip the prompt for trivial edits that already comply.

5. **Key requirements:**
   - Always use MUI components (`@mui/material`)
   - Import colors from `@/theme/tokens`
   - No inline hex colors or arbitrary spacing
   - Extract page logic into custom hooks
