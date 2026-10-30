[🇰🇷 한국어](ko/frontend.md)

# Frontend Tech Stack

## Core Framework

| Technology | Note |
|-----------|------|
| Next.js | App Router |
| React | 19 |
| TypeScript | 6 |

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

Shared route components consume artist configuration for themes, navigation, and metadata. Public visibility is separate from backend availability: prepared configurations do not become accessible or indexable until explicitly made public. The landing page loads availability on the server; unknown or non-public artist routes return 404.

---

## Result Experience

Matched results show the song and related recommendations. No-match and exhausted-question responses also include suggestions; an exhausted quiz lets the user reopen the last question or continue to a no-match result.

Shared results load on the client through the same-origin API route after persisted language state is hydrated. Language query values are validated and malformed legacy links are normalized at the boundary. Results remain available after quiz state expires, although expired sessions no longer provide answer history.

---

## Deployment

**Cloudflare Workers** via [OpenNext](https://opennext.js.org/) adapter.

- Separate dev and production workers
- Backend API proxied through frontend routes to avoid CORS issues

---

## Request Reliability

The browser API client retries eligible reads for transient failures with bounded delays. Mutations do not retry by default. Browser and Worker fetch wrappers abort timed-out requests and clean up timers and cancellation listeners. This complements backend conflict detection and idempotent result creation.

## Responsive UI & Accessibility

Shared artist screens use responsive layouts, keyboard controls, and ARIA labels. Focused Zustand subscriptions reduce unrelated renders, while explicit loading and error states cover result loading and session recovery.

---

## Testing & Quality

Jest + React Testing Library for unit tests across hooks, components, and the API client. CI enforces coverage thresholds alongside lint, type checking, and build checks. Manual verification targets the full quiz flow, accessibility, and cross-browser behavior.

---

## SEO

- Per-route metadata, `robots.ts`, `sitemap.ts`, and a web manifest
- JSON-LD structured data for Organization and WebApplication

---

## Key Features

- **Sharing**: Copy link or share to X/Twitter after getting a result. Results remain accessible even after session expiry via a persistent fallback mechanism
- **Performance work**: Reduced client boundaries and focused store subscriptions
- **Operational visibility**: Backend request outcome and latency events use operation names instead of raw session URLs; sanitized CSP reports help diagnose policy violations
- **Security**: API secret kept server-side (Cloudflare Worker environment), boundary validation for external input, strict CSP / HSTS / Permissions-Policy response headers
