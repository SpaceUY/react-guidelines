---
title: Guided Tours with driver.js
layout: default
nav_order: 21
---

# Guided Tours with driver.js

Some screens end up with enough non-obvious fields or actions that new users need a nudge — but building a custom overlay/highlight engine for that is a distraction from the actual feature. We've standardized on **[driver.js](https://driverjs.com)** for this: a lightweight, framework-agnostic library that highlights real DOM elements and walks the user through them with popovers, without shipping its own React bindings or opinions about your app.

> **Last verified with** `driver.js@1.8.0`.

## What driver.js is

- A **vanilla JS library**, no React/Vue/etc. lock-in — we wrap it ourselves in a small hook so it fits our component model.
- It operates on **live DOM elements via CSS selectors** — no virtual overlay system to keep in sync, no portal management on our side.
- Ships **one small CSS file** you own and can restyle completely — no shadow DOM, no CSS-in-JS conflicts.

## When to use it

Reach for a tour when a screen has several fields or actions that aren't self-explanatory from labels alone — not as a replacement for good UX copy or empty states.

The pattern we've converged on: **one tour per screen, manually triggered** by a visible "Help" button — no auto-start on first visit, no persistence of "already seen." This is a deliberate simplicity choice, not a hard limitation of the module: if a project genuinely needs auto-start for first-time users, that's an extension to design (e.g. a `localStorage` flag checked on mount), not something to bolt on without thinking through the UX first.

## Architecture: two layers

Keep the library-facing code and the business content in separate places:

1. **Reusable core module** — a hook, a button component, shared config, and a type alias. No project-specific logic. Portable as-is to another project.
2. **Per-screen step definitions** — one `<screen>.tour.ts` file next to each screen that needs a tour, exporting the array of steps for that screen only.

```
src/common/tours/
├── index.ts          # public exports
├── use-tour.ts        # useTour hook
├── tour-button.tsx     # TourButton component
├── driver-config.ts    # DEFAULT_TOUR_CONFIG
├── types.ts            # TourStep type
└── tour-theme.css       # popover theming

src/modules/user-profile/
├── user-profile.tour.ts        # steps for this screen
└── components/
    └── user-profile-form.tsx    # consumes TourButton + the steps above
```

## Setup

### 1. Installation

```bash
pnpm add driver.js
```

### 2. Global CSS

Import driver.js's base styles plus your custom theme once, at the app's entry point:

```ts
// src/main.tsx
import 'driver.js/dist/driver.css'
import '@/common/tours/tour-theme.css'
```

### 3. The reusable core module

`TourStep` is a direct alias of driver.js's own step type — no extra abstraction on top of it:

```ts
// src/common/tours/types.ts
import type { DriveStep } from 'driver.js'

export type TourStep = DriveStep
```

Shared defaults for every tour in the app — button labels, progress text, overlay behavior — live in one place so tone and UX stay consistent screen to screen:

```ts
// src/common/tours/driver-config.ts
import type { Config } from 'driver.js'

export const DEFAULT_TOUR_CONFIG: Partial<Config> = {
  animate: true,
  smoothScroll: true,
  showProgress: true,
  progressText: 'Step {{current}} of {{total}}',
  nextBtnText: 'Next',
  prevBtnText: 'Previous',
  doneBtnText: 'Done',
  overlayOpacity: 0.6,
  stagePadding: 4,
  skipMissingElement: true,
}
```

> The button/progress text above is placeholder English — see [Text and i18n](#text-and-i18n) before copying it into a real project.

The hook creates a fresh driver.js instance per call and starts it — it's intentionally not stateful (no progress tracking, no `stop`/`destroy` exposed):

```ts
// src/common/tours/use-tour.ts
import { driver } from 'driver.js'
import { DEFAULT_TOUR_CONFIG } from './driver-config'
import type { TourStep } from './types'

function stepsReachableNow(steps: TourStep[]): TourStep[] {
  return steps.filter((step) => {
    if (typeof step.element !== 'string') return true
    return document.querySelector(step.element) !== null
  })
}

export function useTour(steps: TourStep[]) {
  const start = () => {
    driver({ ...DEFAULT_TOUR_CONFIG, steps: stepsReachableNow(steps) }).drive()
  }

  return { start }
}
```

See [Handling conditional elements](#handling-conditional-elements) for why `stepsReachableNow` exists.

The button is the only thing screens actually import — it wraps `useTour` so no screen calls the hook directly:

```tsx
// src/common/tours/tour-button.tsx
import { CircleHelp } from 'lucide-react'
import { Button } from '@/components/ui/button'
import { Tooltip, TooltipContent, TooltipTrigger } from '@/components/ui/tooltip'
import { useTour } from './use-tour'
import type { TourStep } from './types'

type TourButtonProps = {
  steps: TourStep[]
  label?: string
  disabled?: boolean
}

export function TourButton({ steps, label = 'View this screen’s tour', disabled = false }: TourButtonProps) {
  const { start } = useTour(steps)

  return (
    <Tooltip>
      <TooltipTrigger asChild>
        <Button variant='outline' size='sm' disabled={disabled} onClick={start}>
          <CircleHelp className='mr-2 size-4' />
          Help
        </Button>
      </TooltipTrigger>
      <TooltipContent>{label}</TooltipContent>
    </Tooltip>
  )
}
```

### 4. Marking target elements

Add a `data-tour` attribute to every JSX element a step should highlight. This is the only change a tour requires in the screen's existing markup:

```tsx
<Input
  data-tour='user-profile-display-name'
  value={displayName}
  onChange={(event) => setDisplayName(event.target.value)}
/>
```

### 5. Defining steps for a screen

One `<screen>.tour.ts` file per screen, exporting a `TourStep[]` that targets the `data-tour` attributes added above:

```ts
// src/modules/user-profile/user-profile.tour.ts
import type { TourStep } from '@/common/tours'

export const userProfileTourSteps: TourStep[] = [
  {
    element: '[data-tour="user-profile-display-name"]',
    popover: {
      title: 'Display name',
      description: 'This is what other members see — it does not have to match your legal name.',
    },
  },
  {
    element: '[data-tour="user-profile-visibility"]',
    popover: {
      title: 'Profile visibility',
      description: 'Controls whether your activity is visible to other members of your team.',
    },
  },
]
```

### 6. Wiring it into the screen

Import the button and the steps, and drop the button in the screen's toolbar or header:

```tsx
// src/modules/user-profile/components/user-profile-form.tsx
import { TourButton } from '@/common/tours'
import { userProfileTourSteps } from '../user-profile.tour'

export function UserProfileForm() {
  return (
    <div className='flex items-center justify-between'>
      <h2>Profile settings</h2>
      <TourButton steps={userProfileTourSteps} />
    </div>
    // ...form fields with the matching data-tour attributes
  )
}
```

That's the whole integration: no provider to mount, no context to thread through the tree.

## Handling conditional elements

A step whose `element` selector isn't in the DOM breaks the tour in a specific way: even with `skipMissingElement: true`, driver.js still counts that step against the total when rendering `"Step X of Y"`, so the counter jumps or the tour stalls on a step nobody sees.

This shows up whenever a step targets a field that only renders under a condition (a section that appears after a checkbox is toggled, a tab that isn't active yet). The fix lives in `useTour`, in `stepsReachableNow`: it filters the step list down to elements actually present in the DOM *right before* the tour starts, so the progress count matches what the user will actually see.

**Rule of thumb:** don't target an element that's inside a closed modal or an inactive tab — it won't exist in the DOM when the tour starts, and `stepsReachableNow` will simply drop that step rather than open the modal for you. If a step genuinely depends on interaction mid-tour (open this modal, now look at this), that's outside what this module does today.

## Text and i18n

The snippets above use plain English strings — that's a placeholder, not a recommendation to hardcode copy. Two real cases:

- **The project has an i18n library** (`react-i18next`, `react-intl`, etc.) — tour copy is UI copy. Every string in `DEFAULT_TOUR_CONFIG` and every `popover.title` / `popover.description` in a `*.tour.ts` file goes through the same translation keys as the rest of the screen. Don't special-case tours as the one part of the UI that skips i18n.
- **The project has no i18n** — write the copy in whatever language the rest of that app's UI is already in. Don't invent a different language convention for tours specifically; match the surrounding screen.

## Theming

driver.js ships a default popover look that rarely matches a project's design system. Rather than living with it, override its CSS classes (`.driver-popover`, `.driver-popover-title`, `.driver-popover-description`, button classes, etc.) in a small stylesheet, imported once alongside the base CSS.

If the project themes components with CSS variables (as with [shadcn/ui]({% link docs/4_styling_library.md %}) and [tweakcn]({% link docs/19_tweakcn_theming.md %})), point the override at those same tokens (`--popover`, `--popover-foreground`, `--primary`, `--radius`, etc.) instead of hardcoding colors — that's what makes the tour follow the app's light/dark theme automatically instead of needing its own theme switch. Projects using a different styling approach should adapt the mechanism (theme-aware values from whatever system they use) — the goal, not the exact CSS variables, is what carries over.

## The rules

1. Split every tour into the reusable core module (hook, button, config, type) and per-screen `*.tour.ts` step files — never inline steps into a component.
2. Trigger tours manually via a visible help affordance. Don't auto-start a tour without a specific, agreed reason.
3. Never target an element that isn't guaranteed to be in the DOM when the tour starts (closed modals, inactive tabs). Rely on `stepsReachableNow` for elements that are conditionally rendered on the same screen.
4. Tour copy follows the same i18n rules as the rest of the app — see [Text and i18n](#text-and-i18n).
5. Theme the popovers to match the project's design tokens; don't ship the default driver.js look into a themed app.

## References

- driver.js — [driverjs.com](https://driverjs.com)
- driver.js GitHub — [github.com/kamranahmedse/driver.js](https://github.com/kamranahmedse/driver.js)
