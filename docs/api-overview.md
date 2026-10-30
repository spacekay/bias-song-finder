[🇰🇷 한국어](ko/api-overview.md)

# API Overview

## Design Philosophy

The quiz requires no user account. A session key identifies progress stored in Redis, while PostgreSQL retains results and session-to-tenant mappings for sharing after session expiry. Application instances do not keep authoritative quiz state in memory.

Browser requests pass through same-origin Next.js API routes. Those routes attach the backend shared secret on the server; the browser never receives it.

---

## Session Lifecycle

- Session creation initializes the candidate pool for the selected artist
- Subsequent quiz requests carry the session key
- Sessions have a configurable TTL with explicit extension support
- Ordinary updates preserve the remaining TTL rather than renewing it
- Expired quiz state returns `410 Gone`; persisted result retrieval remains available

### Concurrent Updates

Session mutations atomically compare a version before saving. A stale update or repeated answer returns `409 Conflict` with `SESSION_CONFLICT`, preventing one request from overwriting another answer. A caller must refresh current state before submitting a new answer rather than blindly retrying the old mutation.

---

## Quiz Flow

1. **Create a session** — initialize the artist's candidate pool
2. **Retrieve a question** — select a question that differentiates the remaining songs
3. **Submit an answer** — filter candidates and return the next outcome
4. **Finish or recover** — show a match, offer related songs, or reopen the last question after exhaustion

Most questions support “don't know” without filtering candidates. Keyword questions use a “None of these” option instead of a separate skip button; this also advances without filtering.

### Dynamic Question Strategy

The engine moves from broad filtering to targeted differentiation as the pool shrinks. Lyric hints and direct song selection help resolve small candidate pools. Album options distinguish original releases from repackages, while member and solo-song rules avoid questions that cannot meaningfully differentiate candidates.

### Answer Outcomes

| Status | Behavior |
|--------|----------|
| `CONTINUE` | Continue the question loop |
| `RESULT` | Return a matched song and related recommendations inside `finalResult` |
| `NO_MATCH` | Return suggestions in the answer response's `similarSongs` field |
| `ALL_QUESTIONS_EXHAUSTED` | Return suggestions and preserve a recoverable last question |

Answer responses and persisted result responses have distinct contracts. Next-question retrieval can still return an error for empty or exhausted flows; the recoverable outcomes above describe answer submission.

### Reopening the Last Question

After exhaustion, `POST /api/v1/sessions/reopen-last-question` cancels the last answer and returns that question for another selection. The candidate pool and derived filtering state are restored to the corresponding state before the answer. If no question can be reopened, the API returns a session conflict. The user can alternatively continue to a no-match result.

### Result Enrichment and Sharing

When a song is matched, supported skipped questions receive that song's actual values in the result payload. The result can explain answers the user did not know.

`GET /api/v1/results/{sessionKey}` retrieves the persisted outcome even after Redis expiry. Recommendations can be recomputed from the saved song; answer history is available only while the session survives.

---

## Result Consistency

Result creation is serialized by session within a database transaction. A partial unique index allows only one active result per session. Repeated or concurrent final-result and no-match creation requests keep the first active result, avoiding duplicate song associations and creation notifications.

---

## Response and Security Boundaries

- Success responses return data directly with standard HTTP success codes
- Errors return a structured code and message, including conflicts and expiry
- Worker API routes attach the shared secret to backend requests
- Quiz and shared-result endpoints use session keys without user login
- JWT support carries authenticated context and tenant claims where used; it is not a prerequisite for the browser quiz flow
- Distributed rate limiting constrains per-session requests

## Multi-Tenancy

Each artist has an isolated database schema. Tenant resolution uses authenticated context, active session data, or a persistent session-to-tenant mapping for expired shared links. Resolved identifiers are validated before connection routing, and connections are reset before reuse.
