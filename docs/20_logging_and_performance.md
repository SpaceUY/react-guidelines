---
title: Logging & Monitoring
layout: default
nav_order: 20
---

# Logging & Monitoring with PostHog

Hey team! Every project eventually needs to answer two questions: _"is it broken?"_ and _"what were people actually doing when it broke?"_ We've standardized on **[PostHog](https://posthog.com)** to answer both — error tracking, session replay, and product analytics come from one SDK and one dashboard instead of stitching three tools together.

Sentry is still a perfectly good error tracker, and some clients mandate it. When that happens, jump to [When a client requires Sentry](#when-a-client-requires-sentry) at the end of this doc — everything before that section assumes PostHog.

> **Scope.** This standard covers **React browser observability**: frontend exceptions, browser logs, product events, session replay, and real-user performance. Backend instrumentation and infrastructure uptime are separate setups and are not covered here.
>
> **Last verified with** `posthog-js@1.368.0`, `@posthog/react@1.10.3`.

## Why PostHog

- **One SDK, one dashboard.** Exceptions, session replays, logs, and product events land in the same project, so an error links straight to the replay of the session that produced it.
- **Analytics we'd need anyway.** Error tracking alone doesn't tell us whether a feature is used. PostHog covers both, so we don't bolt on a second vendor later.
- **Generous free tier.** Self-hosting remains technically possible for strict infrastructure requirements, but PostHog explicitly does not support it — their docs state you "assume all responsibility and risk" and that they don't offer customer support for self-hosted instances. **Prefer PostHog Cloud EU when EU data residency is the actual requirement.**
- **Feature flags and experiments** ship in the same SDK — out of scope for this doc, but worth knowing they're there before reaching for another dependency.

## Setup

### 1. Installation

Two packages: the core JS SDK plus the React bindings.

```bash
pnpm add posthog-js @posthog/react
```

### 2. Environment variables

Never hardcode the project token. Add it to `.env.local` (git-ignored) and to the CI/CD variables for each environment:

```env
VITE_POSTHOG_PROJECT_TOKEN=phc_xxxxxxxxxxxxxxxxxxxx
VITE_POSTHOG_HOST=https://us.i.posthog.com
```

Use `https://eu.i.posthog.com` for EU-hosted projects. The project token is a public, write-only key — it's safe in the client bundle — but keeping it in env vars is what lets one build target dev, staging, and production without code changes.

### 3. Provider setup

Wrap the app once, at the root. Passing `apiKey` + `options` lets `PostHogProvider` initialize the client for you, so there's no `posthog.init()` call and no import-order problem to reason about.

**Inside React components, always reach PostHog through `usePostHog()`** — never by importing the `posthog` singleton. PostHog's docs warn that a direct import "will likely cause errors as the library might not be initialized yet." For framework-independent modules (an Axios interceptor, a plain utility) where a hook isn't available, initialize the singleton yourself and hand it to the provider via its `client` prop instead — that's a supported alternative to `apiKey`/`options`, and it makes the initialization order explicit rather than accidental.

```tsx
// src/main.tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { PostHogProvider } from "@posthog/react";
import App from "./App";

const options = {
  api_host: import.meta.env.VITE_POSTHOG_HOST,
  // Opts into the current recommended defaults (autocapture, pageviews, web vitals).
  // Pinned as a date so a future SDK release can't silently change behavior.
  defaults: "2026-05-30",
  // Capture unhandled errors and promise rejections
  capture_exceptions: true,
  // Structured logs — see "Structured logging" below
  logs: {
    serviceName: "my-app-web",
    environment: import.meta.env.MODE,
    serviceVersion: import.meta.env.VITE_APP_VERSION,
  },
} as const;

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <PostHogProvider
      apiKey={import.meta.env.VITE_POSTHOG_PROJECT_TOKEN}
      options={options}
    >
      <App />
    </PostHogProvider>
  </StrictMode>,
);
```

Only initialize in environments where you want data. A common pattern is to render `PostHogProvider` conditionally, or to point local development at a separate PostHog project so dev noise never pollutes production metrics.

### 4. Source maps (Vite)

Without source maps, production stack traces point at minified bundles and are useless. PostHog ships a Rollup plugin that Vite picks up directly — see [Project Setup with Vite]({% link docs/16_project_setup.md %}) for the base config this slots into.

```bash
pnpm add -D @posthog/rollup-plugin
```

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import posthog from "@posthog/rollup-plugin";

export default defineConfig({
  build: { sourcemap: true },
  plugins: [
    react(),
    posthog({
      personalApiKey: process.env.POSTHOG_API_KEY,
      projectId: process.env.POSTHOG_PROJECT_ID,
      host: process.env.POSTHOG_HOST,
      sourcemaps: {
        enabled: true,
        releaseName: "my-app",
        releaseVersion: process.env.GIT_COMMIT_SHA,
        // Strip maps from the deployed bundle after upload
        deleteAfterUpload: true,
      },
    }),
  ],
});
```

`POSTHOG_API_KEY` here is a **personal API key with write access** — unlike the project token, this one is a real secret. It belongs in CI variables only, never in a committed `.env`. Keep `deleteAfterUpload: true` so maps reach PostHog but aren't served publicly.

Setting `releaseVersion` to the commit SHA is what ties an error back to a specific deploy.

### 5. Verify the integration

Don't assume it works because it compiles. Walk this list once per project, against a real deployed environment:

1. **Test event** — trigger any action and confirm it lands in the PostHog activity feed.
2. **Deliberate exception** — throw from a button handler and confirm an issue appears in Error tracking.
3. **Symbolication** — confirm that issue's stack trace shows your source, not minified bundle output. If it shows `chunk-A1B2C3.js:1:48210`, source-map upload isn't working.
4. **Session replay** — find the replay attached to that issue and confirm sensitive fields are masked.
5. **Logout** — log out, log back in as a different user, and confirm the two sessions are separate people rather than one merged identity.

**If events silently don't arrive, check Content Security Policy first.** A restrictive `script-src`, `connect-src`, or `worker-src` will block ingestion or replay while the integration looks perfectly installed in code — no errors in your app, just no data. Confirm the PostHog host is allowlisted in all three directives; replay in particular needs `worker-src`.

## Error Tracking & Logging

### 1. Automatic exception capture

With `capture_exceptions: true`, PostHog captures unhandled errors and unhandled promise rejections out of the box. The granular options let you tune that:

```ts
capture_exceptions: {
	// Uncaught errors via window.onerror
	capture_unhandled_errors: true,
	// Unhandled promise rejections
	capture_unhandled_rejections: true,
	// console.error calls — off by default, and usually should stay off (noisy)
	capture_console_errors: false,
}
```

### 2. Manual capture

For anything you catch yourself, use `captureException`. The second argument is arbitrary properties that show up alongside the stack trace:

```tsx
// src/components/CheckoutButton.tsx
import { usePostHog } from "@posthog/react";

const CheckoutButton = ({ orderId }: { orderId: string }) => {
  const posthog = usePostHog();

  const handleCheckout = async () => {
    try {
      await processOrder(orderId);
    } catch (error) {
      posthog.captureException(error, {
        component: "CheckoutButton",
        orderId,
      });
      // Show a user-friendly error or fallback
    }
  };

  return <button onClick={handleCheckout}>Checkout</button>;
};
```

> **Never** send exceptions with `posthog.capture("$exception", { ... })`. PostHog's docs call this out specifically: only `captureException` runs stack-trace processing, so a hand-rolled `$exception` event arrives unsymbolicated and ungrouped.

### 3. Error boundary

PostHog captures unhandled exceptions with or without error boundaries, but boundaries still earn their place:

- Prevents the entire application from crashing when one component throws
- Provides a graceful fallback UI for users
- Captures React-specific rendering errors with proper component context
- Limits the blast radius to the affected component tree

`@posthog/react` exports **`PostHogErrorBoundary`**, which reports to PostHog automatically:

```tsx
// src/App.tsx
import { PostHogErrorBoundary } from "@posthog/react";
import MainContent from "./components/MainContent";

const FallbackComponent = () => (
  <div className="error-boundary">
    <h2>Something went wrong</h2>
    <p>We've been notified and are working on a fix</p>
    <button onClick={() => window.location.reload()}>Refresh the page</button>
  </div>
);

const App = () => (
  <PostHogErrorBoundary fallback={<FallbackComponent />}>
    <MainContent />
  </PostHogErrorBoundary>
);

export default App;
```

`fallback` also accepts a component rather than an element, in which case it receives `{ error, exceptionEvent, componentStack }` — use that when the fallback needs to show a support reference or branch on the error. The `additionalProperties` prop (an object or a function of the error) tags everything the boundary reports, which is handy for marking which feature area a boundary guards:

```tsx
<PostHogErrorBoundary
  fallback={CheckoutFallback}
  additionalProperties={{ feature: "checkout" }}
>
  <CheckoutFlow />
</PostHogErrorBoundary>
```

Prefer several boundaries around error-prone subtrees (a dashboard widget, a payment step) over one boundary at the root — a single top-level boundary turns any error into a blank page, which is exactly what we're trying to avoid.

### 4. Structured logging

Don't reach for `capture()` to record diagnostics. PostHog draws a deliberate line here, and so should we:

| | Use | For |
|---|---|---|
| **Product event** | `posthog.capture("order_completed", { ... })` | What the **user** did — feeds funnels, retention, and insights |
| **Log** | `posthog.logger.info("payment retry", { ... })` | What the **system** did — feeds the Logs interface, searchable by service and severity |
| **Exception** | `posthog.captureException(error, { ... })` | Something broke — feeds issue grouping and symbolication |

Sending diagnostics as product events (`capture("app_log", { level: "warn" })`) pollutes your analytics with system noise and gives you no real severity filtering. `posthog.logger` emits proper OpenTelemetry log records instead, with structured attributes and the service metadata from the `logs` config in the provider options:

```ts
posthog.logger.debug("cache miss", { key: "user:123" });
posthog.logger.info("checkout completed", { order_id: "ord_789", total_cents: 4200 });
posthog.logger.warn("payment retry", { attempt: 2, provider: "stripe" });
posthog.logger.error("webhook signature invalid", { provider: "stripe" });
```

Requires `posthog-js@1.368.0` or later. Beyond `serviceName` / `environment` / `serviceVersion`, the `logs` config also takes `resourceAttributes`, `flushIntervalMs` (default 3000), `maxBufferSize` (default 100 records), and `maxLogsPerInterval` (default 1000) — that last one is a rate limit, so a hot loop calling `logger.debug` will silently drop records rather than flooding.

**Be deliberate about volume.** Logs and events are both billed on ingestion. Debug-level logging on every render is how a project discovers its PostHog bill. Keep `debug` out of production, and don't log inside render paths or high-frequency handlers.

### 5. Standardized reporting utility

Wrapping the three primitives in one utility keeps reporting consistent, applies shared context automatically, and pairs remote reporting with local console logging so developers see the same signals during development:

```ts
// src/utils/reporting.ts
import type { PostHog } from "posthog-js";
import logger from "./logger";

type ReportLevel = "error" | "warn" | "info" | "debug";

interface ReportContext {
  component: string;
  endpoint?: string;
  extra?: Record<string, unknown>;
}

const toAttributes = (context: ReportContext) => ({
  component: context.component,
  ...(context.endpoint && { endpoint: context.endpoint }),
  ...context.extra,
});

export const createReporter = (posthog: PostHog) => ({
  /**
   * Something broke — goes to error tracking for grouping and symbolication
   */
  captureError: (error: unknown, context: ReportContext) => {
    posthog.captureException(error, toAttributes(context));
    logger.error(error, context.component);
  },

  /**
   * Diagnostic message — goes to PostHog Logs, not to product analytics
   */
  log: (message: string, level: ReportLevel, context: ReportContext) => {
    posthog.logger[level](message, toAttributes(context));
    logger[level](message, context.component);
  },
});
```

Expose it through a hook so components never touch the client directly:

```ts
// src/hooks/useReporter.ts
import { useMemo } from "react";
import { usePostHog } from "@posthog/react";
import { createReporter } from "../utils/reporting";

export const useReporter = () => {
  const posthog = usePostHog();
  return useMemo(() => createReporter(posthog), [posthog]);
};
```

Usage:

```tsx
// src/components/UserProfile.tsx
import { useReporter } from "../hooks/useReporter";

const UserProfile = ({ userId }: { userId: string }) => {
  const reporter = useReporter();

  const fetchUserData = async () => {
    try {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) {
        throw new Error(`Failed to fetch user: ${response.statusText}`);
      }
      return await response.json();
    } catch (error) {
      reporter.captureError(error, {
        component: "UserProfile",
        endpoint: `/api/users/${userId}`,
        extra: { userId },
      });
      return null;
    }
  };

  // ...
};
```

This keeps error reporting consistent across the app while preserving detailed context about where and when errors occur.

## Identifying Users & Capturing Events

Error tracking is far more useful when you know _who_ hit the error and _what_ they were doing. These are the primitives:

```ts
const posthog = usePostHog();

// After login — links this browser's history to a stable user id
posthog.identify(user.id, {
  email: user.email,
  plan: user.plan,
});

// Update person properties later without re-identifying
posthog.setPersonProperties({ plan: "pro" });

// B2B: associate the user with an organization
posthog.group("company", company.id, { name: company.name });

// Properties attached to every subsequent event
posthog.register({ app_version: import.meta.env.VITE_APP_VERSION });

// Custom events
posthog.capture("order_completed", {
  order_id: order.id,
  total: order.total,
  item_count: order.items.length,
});
```

**Always call `posthog.reset()` on logout.** Skip it and the next person to use that browser gets merged into the previous user's identity — which quietly corrupts both your analytics and your error attribution:

```ts
const handleLogout = async () => {
  await api.logout();
  posthog.reset();
};
```

Two conventions so events stay queryable six months from now:

- Event names in `snake_case`, past tense, `noun_verb` — `order_completed`, not `CompleteOrder`.
- Put the variable part in properties, not the name. `checkout_step_viewed` with `{ step: 2 }` beats `checkout_step_2_viewed`.

Feature flags, A/B experiments, and surveys also come from this same SDK (`useFeatureFlagEnabled`, `PostHogFeature`) — out of scope here, but check them before adding another dependency for that job.

## Session Replay

Replay is how you reproduce a bug you can't reproduce locally. Enable it in the provider options — and configure masking at the same time, not later:

```ts
// src/main.tsx — inside `options`
disable_session_recording: false,
session_recording: {
	// Mask every input by default; opt specific ones back in
	maskAllInputs: true,
	maskInputOptions: {
		password: true,
		email: true,
		text: false,
	},
	// Mask text inside anything matching this selector
	maskTextSelector: "[data-private]",
},
```

Extra controls worth knowing:

- **`ph-no-capture` CSS class** — add it to any element to exclude it from recordings entirely. The pragmatic choice for payment forms, PII panels, and document previews. **Know the side effect:** this class is also in autocapture's default `css_selector_ignorelist`, so clicks and interactions inside that element disappear from product analytics too. Usually that's exactly what you want on a payment form — but if you need the interaction data and only want the pixels hidden, reach for `maskTextSelector` instead.
- **`maskInputFn`** — custom per-element masking logic, e.g. redact only values that look like email addresses.
- **`maskCapturedNetworkRequestFn`** — strip tokens and PII from captured request/response payloads.
- **`maskTextSelector: "*"`** masks all text (inputs excluded) — the nuclear option for high-sensitivity apps.

Masking runs **in the browser**, so masked data is never sent over the network to PostHog. That's the property that makes replay defensible in a privacy review.

## Performance Monitoring

Three different things get called "performance," and PostHog covers them unevenly. Be precise about which one you need:

**Browser real-user metrics — covered.** With the recommended `defaults`, PostHog captures a `$web_vitals` event carrying LCP, CLS, INP, and FCP per pageview. That covers the metrics Google grades us on, broken down by page, browser, and device in the dashboard, with no code from us.

**Backend distributed tracing — covered, but alpha.** PostHog ingests traces over OTLP using standard OpenTelemetry SDKs (no PostHog packages needed on the backend) and lets you search and explore them in the UI. It's marked **alpha**, so don't make it a project's only tracing story yet — and it's backend instrumentation, which puts it outside this doc's scope.

**React render profiling — not covered.** There is no PostHog equivalent to Sentry's `withProfiler`. If a project genuinely needs component-level render profiling, use React DevTools' profiler during development, or see the [Sentry section](#when-a-client-requires-sentry).

**Custom timing events.** For anything application-specific, measure it and send it as a normal event:

```ts
const checkout = async () => {
  const startedAt = performance.now();

  try {
    await validateCart();
    await processPayment();
    await generateOrder();

    posthog.capture("checkout_completed", {
      duration_ms: Math.round(performance.now() - startedAt),
    });
  } catch (error) {
    posthog.captureException(error, {
      component: "checkout",
      duration_ms: Math.round(performance.now() - startedAt),
    });
  }
};
```

Because these are ordinary events, you can chart p50/p95 duration over time, break it down by plan or region, and alert on regressions — the same tooling as everything else.

**Uptime monitoring is out of scope for PostHog.** Synthetic checks and availability alerting belong at the infrastructure layer (CloudWatch, Better Stack, or whatever the project already uses). Don't try to infer uptime from error rates.

## Using the PostHog Dashboard

**Error tracking** groups exceptions into issues by stack trace, showing frequency, number of users affected, first/last seen, and the release that introduced them. From an issue you can:

- Read the symbolicated stack trace (assuming source maps were uploaded)
- Jump to the **session replay** of a session where it occurred — the single biggest reason we're on PostHog
- Filter by environment, release, browser, and device
- Assign the issue, or mark it resolved / suppressed so it stops paging you

**Filtering by environment** requires that you send one. Register it as a super property at startup so every event and exception carries it:

```ts
posthog.register({ environment: import.meta.env.MODE });
```

**Alerts** are configured per-insight in the dashboard. The set worth having on every project: spike in exception volume, new issue in the latest release, and a web-vitals regression threshold.

## Best Practices

1. **Upload source maps** on every production build, keyed by commit SHA — an unsymbolicated stack trace is not a bug report.
2. **Identify users after login and `reset()` on logout** — always both.
3. **Send a release version and an environment** with every event, so you can tell "broken in prod since Tuesday's deploy" from background noise.
4. **Configure replay masking before enabling replay**, not after the first privacy complaint.
5. **Use a separate PostHog project for development** so local noise stays out of production metrics.
6. **Keep `capture_console_errors` off** unless you have a specific reason — it's the fastest way to drown the issue list.
7. **Suppress or fix noisy third-party errors** (browser extensions, ad blockers) rather than learning to ignore the list.
8. **Agree on event naming before shipping the first event.** Renaming events later means rebuilding every insight that used them.
9. **Use `posthog.logger` for diagnostics and `capture()` for user behavior** — don't send logs as product events.
10. **Access PostHog through `usePostHog()` in components.** Direct singleton access is acceptable only in framework-independent modules where you control initialization order.
11. **Budget for volume before turning on debug logging.** Logs and events are billed on ingestion, and `maxLogsPerInterval` silently drops the overflow.
12. **Run the [verification checklist](#5-verify-the-integration)** on a real deployed environment, not just locally.

## Security Considerations

1. **Never send PII without a policy.** Emails, names, and addresses in event properties become a data-retention obligation.
2. **The personal API key used for source-map upload is a real secret** — CI variables only. The project token is public and safe in the bundle; don't conflate the two.
3. **Mask aggressively in replay**, and use `ph-no-capture` on any element that renders sensitive data.
4. **Sanitize captured network payloads** with `maskCapturedNetworkRequestFn` — auth headers and tokens should never leave the browser.
5. **Check data residency** before setup. Use **PostHog Cloud EU** when the client requires EU residency — self-hosting is unsupported and shouldn't be the first answer. Migrating regions later is painful.
6. **Don't put PII in log attributes either.** `posthog.logger` records are as retained and as searchable as events; the same discipline applies.
7. **Respect consent.** Where a cookie banner applies, gate initialization or use PostHog's opt-out controls rather than capturing first and asking later.

## When a Client Requires Sentry

Use Sentry instead of PostHog when a client mandates it, or when joining an existing project already standardized on it. Don't run both for error tracking — pick one, or you'll get two sets of alerts and two half-true dashboards.

The concepts map cleanly; only the API differs.

```bash
pnpm add @sentry/react
```

```tsx
// src/main.tsx
import * as Sentry from "@sentry/react";

Sentry.init({
  dsn: import.meta.env.VITE_SENTRY_DSN,
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration({
      maskAllText: true,
      blockAllMedia: true,
    }),
  ],
  // Performance monitoring sample rate — 10% of transactions
  tracesSampleRate: 0.1,
  // Replay: 10% of all sessions, 100% of sessions with an error
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
  release: import.meta.env.VITE_APP_VERSION,
  environment: import.meta.env.MODE,
});
```

Error boundary — same rationale as [above](#3-error-boundary), different import:

```tsx
<Sentry.ErrorBoundary fallback={<FallbackComponent />}>
  <MainContent />
</Sentry.ErrorBoundary>
```

Manual capture and user context:

```ts
try {
  processOrder(orderId);
} catch (error) {
  Sentry.captureException(error, {
    tags: { component: "Checkout", orderId },
    extra: { productDetails },
  });
}

Sentry.setUser({ id: "user-123", email: "user@example.com" });
```

The [standardized reporting utility](#5-standardized-reporting-utility) ports over directly — swap `posthog.captureException` for `Sentry.captureException` and `posthog.logger[level]` for `Sentry.captureMessage(message, level)`, and the rest of the shape stays the same.

**What Sentry gives you that PostHog doesn't:** frontend transaction and span tracing (`Sentry.startSpan`) as a mature, non-alpha product, and a React render profiler (`withProfiler`) for component-level performance. **What you lose:** product analytics, structured logs, and feature flags — so if the project needs any of those, you're back to two vendors.

Note that "PostHog has no distributed tracing" is no longer the differentiator it once was — PostHog ingests OTLP traces too, just in alpha and backend-only. The durable difference is browser-side spans and React render profiling.

Everything in [Best Practices](#best-practices) and [Security Considerations](#security-considerations) still applies; source maps, releases, environments, masking, and PII discipline are tool-agnostic. Sentry adds one knob worth using: `allowUrls` / `denyUrls` keep third-party script errors out of the issue list.

## References

- PostHog React SDK — [posthog.com/docs/libraries/react](https://posthog.com/docs/libraries/react)
- Error tracking — [posthog.com/docs/error-tracking](https://posthog.com/docs/error-tracking)
- Logs (JavaScript install) — [posthog.com/docs/logs/installation/javascript](https://posthog.com/docs/logs/installation/javascript)
- Distributed tracing (alpha) — [posthog.com/docs/tracing](https://posthog.com/docs/tracing)
- Session replay privacy controls — [posthog.com/docs/session-replay/privacy](https://posthog.com/docs/session-replay/privacy)
- Source maps for Vite — [posthog.com/docs/error-tracking/upload-source-maps/vite](https://posthog.com/docs/error-tracking/upload-source-maps/vite)
- Self-hosting caveats — [posthog.com/docs/self-host](https://posthog.com/docs/self-host)
- Sentry for React — [docs.sentry.io/platforms/javascript/guides/react](https://docs.sentry.io/platforms/javascript/guides/react/)
