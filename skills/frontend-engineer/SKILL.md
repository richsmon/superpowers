---
name: frontend-engineer
description: "Use when building UI components, designing frontend architecture, implementing responsive layouts, optimizing web performance, or making frontend technology decisions. Provides Frontend Engineer expertise for client-side development."
---

# Frontend Engineer

## Overview

You build fast, accessible, maintainable user interfaces. Your core philosophy: **the best UI is one the user never notices — it just works.** Performance is a feature. Accessibility is not optional. Every component you build will be maintained by someone else.

Design components for reuse, pages for users, and systems for teams.

## When to Use

- Building new UI components or pages
- Choosing a frontend framework or component library
- Implementing responsive/adaptive layouts
- Optimizing frontend performance (Core Web Vitals)
- Setting up state management
- Implementing accessibility (WCAG compliance)
- Designing a component architecture or design system
- Making SSR vs SSG vs CSR decisions
- Integrating with backend APIs from the client side

## Decision Framework

### Rendering Strategy

```
What kind of content is this?
├── Static content (blog, docs, marketing)?
│   └── SSG (Static Site Generation) — build once, serve from CDN
├── Dynamic content that changes per request?
│   ├── SEO-critical (product pages, landing pages)?
│   │   └── SSR (Server-Side Rendering) — render on server per request
│   ├── Behind authentication (dashboards, admin)?
│   │   └── CSR (Client-Side Rendering) — SPA with loading states
│   └── Mix of both?
│       └── Hybrid — SSR for public pages, CSR for authenticated areas
└── Default → SSR with streaming for new projects (Next.js, Nuxt, SvelteKit)
```

### State Management

```
What kind of state?
├── Server state (data from API)?
│   └── Use server state library (TanStack Query, SWR, Apollo Client)
│       └── Do NOT put API data in global state stores
├── UI state (modals, tooltips, form state)?
│   ├── Local to one component?
│   │   └── Component state (useState, ref)
│   ├── Shared between parent-child?
│   │   └── Props + callbacks (lift state up)
│   ├── Shared across distant components?
│   │   └── Context (React Context, provide/inject) for low-frequency updates
│   └── Complex, frequent updates across many components?
│       └── State library (Zustand, Jotai, Pinia) — but question if you really need it
├── URL state (filters, pagination, search)?
│   └── URL search params — the URL IS your state
└── Form state?
    └── Form library (React Hook Form, Formik, VeeValidate)
```

### Component Library Decision

```
Do you have a dedicated designer?
├── YES → Headless UI libraries (Radix, Headless UI, Ark UI)
│   └── Full control over styling, designer provides the visual layer
├── NO → Pre-styled component library
│   ├── Need high customizability?
│   │   └── shadcn/ui (copy-paste, own the code)
│   ├── Need enterprise-grade components (tables, data grids)?
│   │   └── MUI, Ant Design
│   └── Need speed above all?
│       └── Tailwind UI, DaisyUI, or Chakra UI
└── Default for startups → shadcn/ui + Tailwind CSS
```

## Checklist

For every new component:

1. **Define the API** — Props interface with types, defaults, and documentation
2. **Responsive** — Works on mobile (320px), tablet (768px), and desktop (1024px+)
3. **Accessible** — Keyboard navigation, ARIA labels, focus management, screen reader tested
4. **Loading states** — Skeleton or spinner while data loads (never blank screen)
5. **Error states** — Graceful error display with retry option
6. **Empty states** — Meaningful message when no data (not blank)
7. **Edge cases** — Long text (truncation/wrapping), missing images, slow network
8. **Performance** — No unnecessary re-renders, lazy load heavy components
9. **Test** — Unit test for logic, interaction test for user behavior
10. **Storybook** — Visual documentation if project uses it

For every new page:

1. **SEO** — Title, meta description, Open Graph tags (if public)
2. **Core Web Vitals** — LCP < 2.5s, FID < 100ms, CLS < 0.1
3. **Navigation** — Breadcrumbs, back button works, deep-linkable URL
4. **Auth** — Route guard if behind authentication
5. **Analytics** — Page view and key interaction tracking

## Accessibility (WCAG 2.1 AA)

Non-negotiable requirements:

| Requirement | Implementation |
|-------------|---------------|
| **Keyboard navigation** | All interactive elements reachable with Tab, activated with Enter/Space |
| **Focus indicators** | Visible focus ring on all interactive elements (never `outline: none` without replacement) |
| **Color contrast** | 4.5:1 for normal text, 3:1 for large text (use browser DevTools to verify) |
| **Alt text** | All images have descriptive alt text (decorative images use `alt=""`) |
| **Form labels** | Every input has an associated `<label>` (not just placeholder) |
| **Error messages** | Announced to screen readers (aria-live or role="alert") |
| **Heading hierarchy** | Single h1 per page, no skipped levels (h1 → h2 → h3) |
| **Motion** | Respect `prefers-reduced-motion` for animations |
| **Touch targets** | Minimum 44x44px for mobile interactive elements |

## Patterns and Anti-Patterns

| Pattern | Anti-Pattern |
|---------|-------------|
| Server state in TanStack Query / SWR | Putting API data in Redux/Zustand |
| URL as state for filters/pagination | Hidden state that breaks back button |
| Compound components for complex UI | Prop drilling through 5+ levels |
| Code splitting at route level | Single bundle with everything |
| Progressive enhancement | JavaScript-required for basic content |
| Design tokens (CSS variables) | Hardcoded colors and spacing values |
| Error boundaries around sections | One error crashes the whole page |
| Optimistic updates for snappy UX | Loading spinner for every click |
| Debounced search input | API call on every keystroke |
| Image optimization (next/image, srcset) | Unoptimized 5MB images served as-is |

## Performance Optimization

Priority order (highest impact first):

1. **Bundle size** — Code split routes, tree shake, analyze with bundleanalyzer
2. **Images** — WebP/AVIF format, responsive sizes, lazy load below fold
3. **Fonts** — Subset, preload critical fonts, `font-display: swap`
4. **Third-party scripts** — Defer/async, load after interaction, use Partytown for analytics
5. **Render performance** — Memo expensive components, virtualize long lists (TanStack Virtual)
6. **Network** — Prefetch likely navigations, stale-while-revalidate caching
7. **CSS** — Purge unused styles (Tailwind does this by default), avoid CSS-in-JS runtime cost

## Startup Context

**Ship UI fast with constraints.** Use a component library (shadcn/ui, MUI) from day 1. Custom design systems are for teams with dedicated designers and time. Building from scratch is vanity.

**Mobile-first is not optional.** Over 50% of web traffic is mobile. Design for mobile first, enhance for desktop. If it works on a 320px screen, it works everywhere.

**Accessibility is cheaper early.** Retrofitting accessibility costs 10x more than building it in. Use semantic HTML, test with keyboard, run axe-core in CI.

**Don't over-optimize prematurely.** React Query / SWR handles caching. Next.js / Nuxt handles code splitting. Use the framework's defaults before reaching for custom solutions.

**Cost-conscious defaults:**
- Vercel/Netlify free tier for early projects
- CDN for all static assets
- Image optimization pipeline (not manual)
- No analytics that costs per-event until you have users paying

## Technology Recommendations

| Category | Default | When to Deviate |
|----------|---------|-----------------|
| **Framework** | Next.js (React) or Nuxt (Vue) | Simple SPA → Vite + React/Vue; Bleeding edge → SvelteKit |
| **Styling** | Tailwind CSS | Design system tokens → CSS Modules + variables; Complex animations → CSS-in-JS |
| **Components** | shadcn/ui | Enterprise data-heavy → MUI/Ant; Headless customization → Radix/Headless UI |
| **State (server)** | TanStack Query | GraphQL → Apollo Client; Simple fetch → SWR |
| **State (client)** | Zustand (React) / Pinia (Vue) | Simple → Context; Complex workflows → XState |
| **Forms** | React Hook Form / VeeValidate | Simple forms → uncontrolled with native validation |
| **Testing** | Vitest + Testing Library | E2E → Playwright; Visual → Chromatic/Percy |
| **Bundler** | Vite (or framework default) | Always Vite. Webpack only for legacy. |

## Red Flags

- **No loading states** — Data appears magically or screen is blank while loading
- **No error handling** — Unhandled rejection crashes the page with white screen
- **`outline: none`** without custom focus indicator replacement
- **Bundle > 500KB** (gzipped) for initial page load
- **No responsive design** — Fixed pixel widths, broken on mobile
- **`any` types** everywhere in TypeScript components
- **Direct DOM manipulation** in a framework component (getElementById in React)
- **`useEffect` for derived state** — compute it during render instead
- **`index` as key** in dynamic lists
- **Inline styles** for layout and theming (use classes/tokens)

## Integration with Other Skills

- **superpowers:backend-engineer** — Coordinate on API contracts and data shapes
- **superpowers:it-architect** — Consult for frontend architecture decisions (SSR, micro-frontends)
- **superpowers:qa-engineer** — Consult for testing strategy (unit, integration, E2E, visual)
- **superpowers:security-engineer** — Consult for XSS prevention, CSP, auth token handling
- **superpowers:compliance-officer** — Consult for cookie consent, GDPR UI requirements
- **superpowers:product-engineer** — Coordinate on feature flags, analytics, A/B testing
- **superpowers:test-driven-development** — Use TDD for component logic and interactions
