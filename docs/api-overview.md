# API Overview

## Design Philosophy

The API is designed around a **stateless session flow** — no user login required. A session key acts as both the user identifier and the quiz state pointer.

All quiz state is stored externally, keyed by session key. The backend itself holds no in-memory state, making it horizontally scalable.

---

## Session Lifecycle

- A unique session key is generated on session creation
- The session key is used for all subsequent requests during the quiz
- Sessions have a configurable TTL with explicit extension support
- Expired session requests return an appropriate error status

---

## Quiz Flow

The quiz follows a session-based conversational pattern:

1. **Session creation** — initializes the candidate pool from the artist's full catalog
2. **Question retrieval** — backend dynamically selects the next most-differentiating question
3. **Answer submission** — filters the candidate pool based on the user's answer
4. **Result** — when candidates are narrowed sufficiently, the quiz concludes with the matched song(s)

"Skip" (don't know) is always available — it does not filter candidates, allowing the quiz to proceed even with partial knowledge.

### Dynamic Question Strategy

The question selection algorithm adapts its strategy based on the current candidate pool size, transitioning from broad filtering questions to more targeted differentiation as the pool shrinks. When candidates are few enough, the user can directly select from the remaining options.

---

## Response Format

- Success responses return data directly with standard HTTP success codes
- Error responses return a structured error object with an error code and message

---

## Security Model

- Frontend authenticates with the backend to obtain a short-lived JWT
- The JWT is attached to all requests from the frontend
- Quiz session endpoints use the session key as sufficient authentication
- Distributed rate limiting prevents abuse on a per-session basis

---

## Multi-Tenancy

Each artist group is a separate tenant with its own database schema. Tenant routing is resolved transparently from the session context — from the API consumer's perspective, all endpoints look the same.
