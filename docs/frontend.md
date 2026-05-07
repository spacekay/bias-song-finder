[🇰🇷 한국어](ko/frontend.md)

# Frontend Tech Stack

## Core Framework

| Technology | Note |
|-----------|------|
| Next.js | App Router |
| React | 19 |
| TypeScript | — |

---

## UI & Styling

| Technology | Role |
|-----------|------|
| Tailwind CSS v4 | Utility-first styling |
| shadcn/ui | Accessible component primitives |
| Framer Motion | Animations and transitions |

---

## State Management

**Zustand** — lightweight state management with localStorage persistence so state survives page refreshes within the same session.

---

## Internationalization

Custom i18n implementation supporting **Korean** and **English**.

- Translation files organized per artist and locale
- Language toggle available in-app
- All question text, option labels, and UI copy are fully translated

---

## Routing Structure

Artist-based dynamic routing using Next.js App Router. Each artist has its own branded landing page with the artist's colors applied dynamically. Routes cover the landing page, quiz flow, and result display.

The structure follows an artist-segment pattern (e.g. `/shinee`, `/fx`), so adding a new artist is primarily a matter of swapping artist-scoped data, mappings, and theme — not rewriting flow code.

---

## Result Experience

The result page shows the matched song together with a small set of close-match recommendations bundled in the same response. When the quiz can't pin a single song — no-match or all-questions-exhausted paths — the same component renders the closest songs instead, so users never hit a dead end.

---

## Deployment

**Cloudflare Workers** via [OpenNext](https://opennext.js.org/) adapter.

- Separate dev and production workers
- Backend API proxied through frontend routes to avoid CORS issues

---

## Testing & Quality

Jest + React Testing Library for unit tests across hooks, components, and the API client. Manual coverage targets the full quiz flow, accessibility, and cross-browser behavior.

---

## SEO

- Per-route metadata, `robots.ts`, `sitemap.ts`, and a web manifest
- JSON-LD structured data for Organization and WebApplication

---

## Key Features

- **Sharing**: Copy link or share to X/Twitter after getting a result. Results remain accessible even after session expiry via a persistent fallback mechanism
- **Performance**: INP and CLS optimized for Core Web Vitals
- **Security**: API secret kept server-side (Cloudflare Worker environment), input validation on all user-submitted values, strict CSP / HSTS / Permissions-Policy response headers
