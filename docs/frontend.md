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

---

## Deployment

**Cloudflare Workers** via [OpenNext](https://opennext.js.org/) adapter.

- Separate dev and production workers
- Backend API proxied through frontend routes to avoid CORS issues

---

## Key Features

- **Sharing**: Copy link or share to X/Twitter after getting a result. Results remain accessible even after session expiry via a persistent fallback mechanism
- **Performance**: INP and CLS optimized for Core Web Vitals
- **Security**: API secret kept server-side (Cloudflare Worker environment), input validation on all user-submitted values
