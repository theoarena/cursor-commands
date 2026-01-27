---
name: styling
description: Guidelines for styling the frontend application (css, tailwind, sass etc)
---

## Important
- The application uses shadcn/ui (https://ui.shadcn.com) as the default UI component library.
- shadcn/ui is a collection of reusable components built with Radix UI and Tailwind CSS.
- The components are copied into the project, allowing full customization and control.

## Overview
- Tailwind CSS
- shadcn/ui library (https://ui.shadcn.com) for UI components
- Sass
- BEM pattern and design tokens

## Styles Structure (reference)

```
resources/css
├── _index.scss 
├── _config.scss 
├── _base.scss 
├── _theme.scss 
├── components/
│   ├── dropdown.scss
│   ├── pagination.scss
│   ├── table.scss
│   └── ...
├── pages/ // styles specific to each section of the system
│   ├── cotistas.scss
│   ├── hoteis.scss│   
│   └── ...
├── _typography.scss 
├── _utilities_.scss 
└── ...
```


## 🔗 Official References
- **shadcn/ui**: Official docs at https://ui.shadcn.com/docs
- **shadcn/ui components**: `resources/js/components/ui/`
- **shadcn/ui configuration**: `components.json` (at the project root)
- Utility tokens (mixins/breakpoints/rem/BEM helpers): `resources/css/_config.scss`
- Design tokens and themes (CSS variables + .dark): `resources/css/_theme.scss`
- Base layer (body, scrollbars, resets): `resources/css/_base.scss`
- BEM components (btn, card, layout, header, sidebar, form, loading): `resources/css/components/{component-name}.scss`
- Styles specific to each section of the system: `resources/css/pages/{page-name}.scss`
- Utilities (text, flex, glass, shadow, animations, responsive): `resources/css/_utilities.scss`
- Typography definition (size, color, weight, style, variations, etc.): `resources/css/_typography.scss`

## Layers and Conventions
- Import order: theme → base → components → utilities via `index.scss`.
- BEM: `.block`, `&__elem`, `&--mod`.
- Tailwind utility classes with `@apply` inside BEM blocks.

## Design Tokens (summary)
- Defined in `_theme.scss` under `:root` and `.dark`.
- Colors, spacing, shadows, durations, layout variables.

## SCSS Utilities
- Breakpoints and mixins, rem conversion, BEM helpers.

## Component Patterns (BEM + Tailwind)
```scss
@layer components {
  .block {
    @apply /* base */;
    &__elem { @apply /* elem */; }
    &--mod { @apply /* var */; }
  }
}
```

## Usage Guidelines
- Use the official shadcn/ui documentation (https://ui.shadcn.com/docs) and its native features whenever possible.
- shadcn/ui components are located at `resources/js/components/ui/`.
- Add new components with: `npx shadcn@latest add [component-name]`.
- Prefer design system variants (shadcn/ui) and BEM classes when needed.
- shadcn/ui components can be customized directly in code because they are copied into the project.
- Avoid inline styles.
- Use Tailwind for responsiveness, with SCSS mixins when necessary.

## Example

### shadcn/ui Components
```tsx
import { Button } from '@/components/ui/button';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';

<Button variant="default" size="sm">Save</Button>
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>
    Card content
  </CardContent>
</Card>
```

### Custom BEM Components
```tsx
<button className="btn btn--primary btn--sm">Save</button>
<div className="card card--featured">
  <div className="card__header">
    <h3 className="card__title">Title</h3>
  </div>
</div>
```

## Best Practices
- Tokens live in `_theme.scss`.
- No duplicated utilities.
- Use `!important` only when strictly necessary. Avoid it as much as possible.
- Aim for performant, accessible, and consistent styles.

