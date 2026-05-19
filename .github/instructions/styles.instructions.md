---
applyTo: "**/*.scss,**/*.css,**/*.component.html"
description: Styling rules — Tailwind / SCSS, accessibility, design tokens
---

# Styling rules

> Adjust this file to match your project: Tailwind v4 / SCSS / Angular Material.
> The examples below assume **Tailwind v4 + CSS variables** (the most common modern setup).

## Approach

- Utility-first via Tailwind for layout and common patterns
- Component-scoped `.scss` for anything reusable / complex
- Design tokens as CSS variables on `:root` (HSL preferred for theming)
- Dark mode via `.dark` class on `<html>` (not media query — user override possible)

## Tailwind v4

- Config-less — use `@import "tailwindcss";` in global stylesheet
- Theme via `@theme { --color-primary: ...; }` block
- Use `@apply` sparingly — only for genuine reuse, not "I don't want long class lists"

## Component styles

- Default to component-scoped (Angular default `ViewEncapsulation.Emulated`)
- Use `:host { display: block; }` (or `flex`, etc.) — components are inline by default
- `:host-context()` for parent-state-dependent styling
- No `::ng-deep` — find another way (CSS variables, parent class, slotting)

## Required UI conventions

- Every clickable element: `cursor: pointer` (or `cursor-pointer` utility)
- Every modal: a visible close button (`×` icon) in top-right corner, keyboard-dismissable
- Focus ring visible on all interactive elements — never `outline: none` without replacement
- Hit targets: minimum 44×44 px for touch
- Transitions: use `transition-property` not blanket `transition: all`

## Responsive

- Mobile-first — base styles for mobile, `sm:`, `md:`, `lg:` for larger
- Container queries (`@container`) when component-level breakpoints needed
- Test at 320px width (smallest supported)

## Accessibility colors

- WCAG AA contrast minimum (4.5:1 for body text, 3:1 for large text / UI)
- Never use color alone to convey meaning — add icon, text, or pattern

## What to avoid

- `!important` — almost never necessary; if used, comment why
- Inline styles via `[style.x]` for static values — use class
- Magic numbers — use spacing tokens (`1rem`, `0.5rem`) or Tailwind scale
- `position: absolute` without explicit `inset` / coordinates
- Animations longer than 300ms for UI feedback (>300ms feels sluggish)
