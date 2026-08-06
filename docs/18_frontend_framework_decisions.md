---
title: Frontend Framework Decisions
layout: default
nav_order: 18
---

# Frontend Framework Decisions: SPA, TanStack Start, and SSR

Hey team! One of the first calls on a new React project is whether it needs to be a plain Vite SPA or an SSR framework — and if it needs SSR, why we reach for **TanStack Start** instead of Next.js. This doc walks through that decision, plus one case where it gets trickier in practice: Web3/wallet apps.

**TanStack Start**, if you haven't used it yet, is a full-stack, SSR-capable React framework built on Vite, TanStack Router, and Nitro — the same router/query stack we already standardize on for SPAs, just with a server attached.

For the broader "client-side vs server-side" business question, see [Client server side]({% link docs/2_client_server_side.md %}). For local dev setup, see [Project Setup with Vite]({% link docs/16_project_setup.md %}).

## The core standard

> **Does the project need SEO?**
> - Yes → TanStack Start
> - No → Vite SPA

Within each, a second question — **is Web3 / wallet connection needed?** — determines the architecture pattern, not the framework choice.

---

## Web3 and SSR: why they conflict

Web3 is inherently client-side. Wallets live in the browser (`window.ethereum`, injected providers, WalletConnect sessions). SSR's whole value proposition is rendering on the server. These are philosophically at odds, regardless of which SSR framework you pick.

**The classic problems:**
- Hydration mismatches — server renders with no wallet state, client renders with wallet state
- `window is not defined` — wallet SDKs touching browser globals during SSR
- Provider nesting — `WagmiProvider`, `QueryClientProvider`, `RainbowKitProvider` all need to stay client-only, or the app breaks on the server

**TanStack Start's answer:** SSR is opt-in per route, not all-or-nothing. Public/marketing routes (landing page, FAQ, NFT gallery) render SSR as normal; wallet-dependent routes (dashboard, mint, portfolio) are marked `ssr: false` and behave like a plain SPA. Server functions and route `loader`/`beforeLoad` hooks still give SSR'd routes a backend proxy for RPC calls and allowlist gating, without the wallet providers ever needing to touch the server.

---

## When SSR frameworks actually make sense

Beyond SEO, valid reasons to reach for an SSR framework:

- **Egress optimization** — server-to-server calls on the same private network (~1ms vs ~100ms), not metered as egress, can use internal endpoints
- **Streaming and progressive rendering** — the browser starts rendering the shell immediately while slower data fetches complete server-side
- **First contentful paint on public pages** — HTML arrives pre-rendered rather than waiting for JS to boot

**Page transitions** do not meaningfully benefit from SSR over a well-built SPA with TanStack Router or React Router — we already standardize on TanStack Router/Query for data fetching (see [Data fetching]({% link docs/5_data_fetching.md %})), so an SPA isn't giving up much here.

---

## TanStack Start vs Next.js

TanStack Start is our preferred choice for SSR, for reasons that go beyond "no vendor lock-in":

- **Simpler mental model.** Start renders components as normal, interactive React by default and you opt *into* server-only behavior. Next.js's App Router flips that: every component is a Server Component unless you add `'use client'`. For a team not already fluent in RSC, Start's default is the one that matches how React worked before Server Components existed — less to unlearn, fewer surprise client/server boundary bugs.
- **Type safety that actually reaches the server boundary.** Start validates server functions end-to-end, inputs and outputs included. Next.js's Server Actions can receive data at runtime that TypeScript never flagged, because the type-checking stops at the client/server line.
- **A router with compile-time guarantees, not runtime hints.** TanStack Router — the router we already standardize on — gives typed path params and validated, typed search params, with no Next.js equivalent.
- **Caching you can reason about.** Start treats server output as ordinary data, cached through the same TanStack Query patterns we already use for client-side fetching (see [Data fetching]({% link docs/5_data_fetching.md %})). Next.js's multi-layer implicit caching (fetch cache, Router cache, Full Route cache) has a well-documented history of behaving unpredictably even for experienced teams.
- **A standard Node.js server that fits our infra.** Start's output is a normal Node process — it drops straight into whatever AWS compute we're already using (ECS/Fargate, Elastic Beanstalk) with no framework-specific glue. Next.js *can* self-host the same way via `next start`, but several of its flagship features — ISR revalidation, Image Optimization, Edge Middleware — are built and tuned for Vercel's infrastructure; replicating them on our own AWS setup means extra plumbing (custom cache handlers, `sharp` for image processing, etc.) that Vercel gives you for free.

For the full, official breakdown, see TanStack's own [Start vs Next.js comparison](https://tanstack.com/start/v0/docs/framework/react/start-vs-nextjs).

---

## Vite SSG

An overlooked middle ground. For projects where SEO matters but content is known at build time:

```javascript
// vite.config.ts
import prerender from 'vite-plugin-prerender'

export default {
  plugins: [
    prerender({
      routes: ['/about', '/pricing', '/nft/featured']
    })
  ]
}
```

Outputs static HTML per route — crawlable, no server needed, deploys to S3 + CloudFront or any static host/CDN.

**Covers:** routes known at build time (`/about`, `/pricing`, `/blog/post-1`)
**Doesn't cover:** truly dynamic routes (`/nft/:contract/:tokenId` for arbitrary tokens) — those need a server-side metadata proxy, which is out of scope here.

---

## The decision framework

```
Does the project need SEO?
│
├── YES → TanStack Start
│   └── Web3 needed?
│       ├── No  → Full SSR, no wallet concerns
│       └── Yes → SSR public routes; keep wallet UI client-only in the app subtree
│
└── NO → Vite SPA
    └── Web3 needed?
        ├── No  → Pure SPA, static deploy
        └── Yes → Wallet provider at root, no SSR tension
```

---

## Framework comparison summary

| | Vite SPA | TanStack Start | Next.js |
|---|---|---|---|
| SEO / SSR | ❌ (SSG via plugin) | ✅ | ✅ |
| Web3 compatibility | ✅ best | ✅ with `ssr:false` | ⚠️ friction |
| AWS fit | ✅ S3 + CloudFront | ✅ ECS/Fargate, Elastic Beanstalk | ⚠️ friction |
| Vendor lock-in | None | None | Vercel-optimized |
| Best for | Dapps, dashboards | Hybrid SSR apps | Vercel-hosted SSR |
