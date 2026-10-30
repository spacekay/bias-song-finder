[🇰🇷 한국어](ko/engineering-decisions.md)

# Engineering Decisions

The application must keep a quiz coherent when users answer twice, requests overlap, or their memory is incomplete. These decisions address those cases across the frontend, session store, and database.

## Protect Progress from Concurrent Answers

Two requests can read the same quiz state and independently compute a next state. A normal read-modify-write would let the later save erase the earlier answer.

Redis updates use an atomic version comparison and preserve the remaining session TTL. Stale writes and duplicate answers return a conflict. This protects stored progress across application instances, while leaving callers responsible for refreshing state before another answer. Rate limiting reduces request volume; it does not replace concurrency control.

## Make Result Creation Idempotent

Preventing conflicting session updates alone does not prevent two requests from creating a relational result. Final-result and no-match paths can arrive repeatedly or concurrently.

A transaction-scoped database lock serializes creation for a session, and a partial unique index enforces one active result. The first active result is reused, including its existing song associations. This also prevents duplicate creation notifications on those requests. The guarantee is scoped to result creation; it is not a general exactly-once delivery guarantee for external notifications.

## Recover When Questions Run Out

An exhausted question pool can mean an earlier answer was mistaken. Automatically ending the quiz would remove the user's opportunity to correct it.

The engine preserves a recoverable last question and restores the candidate pool and derived filters to the state before that answer. The user can reopen the question or continue to a no-match result with recommendations. This adds a controlled recovery transition without making arbitrary quiz history editable.

## Preserve Contracts During a Framework Upgrade

The Spring Boot 4 migration moved application serialization to Jackson 3. A dependency upgrade can still change persisted session values and API enum representations even when source types look unchanged.

HTTP and Redis mappers explicitly retain enum names, backed by compatibility tests. Libraries that still require Jackson 2 keep an isolated dependency path. This preserves existing wire formats while accepting a temporary second serialization dependency.

## Keep Requests within Clear Boundaries

Browser quiz and result requests use same-origin Worker API routes, which attach backend credentials on the server. Eligible reads have bounded retries; mutations do not retry by default because a timeout does not prove the server did nothing.

Result loading waits for persisted language hydration and validates language query values. Artist configuration separately controls public visibility, so preparing another artist does not immediately expose its routes. Backend availability still determines which public artists can be selected.

## Verification and Operational Visibility

The implementation repositories include service tests, PostgreSQL/Redis integration tests, frontend coverage gates, lint, type checks, and build checks. Backend readiness groups check database, Redis, and migration readiness separately from liveness. Worker events record request outcomes and latency using operation names, and CSP reports are sanitized before logging.

These are implemented checks and diagnostic tools, not evidence of a measured latency improvement or an uptime guarantee.

---

## Source Baseline

Reviewed on **2026-10-30** against local implementation snapshots:

| Repository | Revision | Version source |
|------------|----------|----------------|
| Backend | `976355b` | Gradle version catalog: Kotlin 2.4.21, Spring Boot 4.1.1; Java 21 toolchain |
| Frontend | `c48f5dd` | Package manifest: Next.js 16.4.3, React 19.3.1, TypeScript 6.0.3 |

This portfolio summarizes implementation behavior. It does not establish which revision is currently deployed. Internal ranking weights and operational tuning values are intentionally omitted.
