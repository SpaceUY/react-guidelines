---
title: Theming with tweakcn
layout: default
nav_order: 19
---

# Theming with tweakcn

Hey team! We've standardized on [ShadCN/UI]({% link docs/4_styling_library.md %}) as our component library — but shadcn ships with near-default tokens, and every project that doesn't touch them ends up looking like every other shadcn site. Hand-editing OKLCH color values and spacing scales in a global CSS file to fix that burns hours we don't need to spend.

This doc defines **[tweakcn](https://tweakcn.com)** as our standard for that: a visual, open-source theme editor built specifically for shadcn/ui and Tailwind CSS. It gives us real-time preview over full themes — colors, typography (including font sizes), border radius, spacing, shadows — and exports them as plain CSS variables we own. No SDK, no runtime dependency, no lock-in: if the tool disappears tomorrow, the CSS keeps working.

## What tweakcn is

- A **visual, no-code theme editor** for shadcn/ui on Tailwind CSS — open source ([github.com/jnsahaj/tweakcn](https://github.com/jnsahaj/tweakcn)), free for our core workflow.
- It operates **entirely on shadcn's existing CSS variable contract** (`--primary`, `--background`, `--card`, `--radius`, `--shadow-*`, etc.) — no new abstraction on top of what shadcn already defines.
- **Export = paste.** Output is a `:root` / `.dark` CSS variables block that drops straight into the project's global CSS — a readable, reversible, diffable block in a PR.

## Why we use it: it covers more than color

Beyond palettes, tweakcn gives visual control over every token shadcn exposes:

| Dimension | What you can tune |
|---|---|
| **Color** | Background, foreground, primary/secondary/accent, card, popover, muted, destructive, border, input, ring, sidebar, chart-1..5 — light & dark independently |
| **Radius** | Global `--radius` scale — compare sharp / soft / pill looks across all components instantly |
| **Spacing** | Tailwind spacing properties — preview denser vs. airier layouts |
| **Shadows** | Full shadow stack (color, opacity, blur, spread, offsets) — flat vs. elevated aesthetics |
| **Typography** | Font family, **font size scale**, weight, letter spacing / text transform |

That last row is the one to call out: when a design calls for a different type scale than shadcn's default (denser data tables, larger marketing headings, whatever the project needs), tweakcn is where we explore font sizes visually against real components instead of guessing values in `index.css`. Because everything renders live, "what if this felt more compact / rounder / flatter" becomes a five-minute experiment instead of a refactor.

**Accessibility is built in** — a contrast ratio checker validates text/background pairs against WCAG before a theme ever gets exported. Treat this as a required gate: no theme ships with failing contrast pairs.

## Standard workflow

### New project bootstrap

```bash
npm create vite@latest my-app        # or TanStack Start when SEO matters — see [Frontend Framework Decisions]({% link docs/18_frontend_framework_decisions.md %})
npx shadcn@latest init

# Apply a theme by pasting the exported :root / .dark block into src/index.css
```

### Theme creation / iteration loop

1. Open tweakcn → start from a preset close to the target, or import the project's current theme via **CSS import**.
2. Explore palette → radius → spacing/shadows → typography (including font sizes), checking light **and** dark mode.
3. Run the contrast checker; fix any failing pairs.
4. Export the CSS variables block → commit it to the project's global CSS.
5. Export the JSON preset too → commit alongside it, so the theme is reviewable and reusable across projects.

### Restyling an existing project

tweakcn supports **importing existing theme CSS**, so a live project's variables can be pulled into the editor, remixed, and re-exported — useful for rebrands or "refresh the look" requests without touching component code.

## Limitations

1. **shadcn/Tailwind only.** Projects outside the shadcn track are out of scope for this standard.
2. **A theming layer, not a design system.** It governs tokens (color, radius, spacing, shadows, type) — component variants, iconography, motion, and content/UX guidelines still need our own conventions on top.
3. **Typography controls are lighter** than full design-system tooling — fine for picking a font size scale, not a replacement for a full type system on brand-heavy projects.
4. **AI theme generation is a paid feature** beyond a small free quota — treat it as an optional accelerant, not a dependency.

## The rules

1. Every new shadcn project applies a theme at bootstrap — no project ships stock shadcn tokens.
2. Themes are committed to the repo as **CSS variables** (required) and, where useful, the **tweakcn JSON preset** alongside them.
3. The contrast checker must pass before a theme is exported.
4. Light and dark mode are both defined, always.

## References

- tweakcn — [tweakcn.com](https://tweakcn.com)
- tweakcn GitHub — [github.com/jnsahaj/tweakcn](https://github.com/jnsahaj/tweakcn)
- shadcn/ui Theming — [ui.shadcn.com/docs/theming](https://ui.shadcn.com/docs/theming)
