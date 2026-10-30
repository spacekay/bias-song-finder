[🇰🇷 한국어](ko/backend.md)

# Backend Tech Stack

## Core Framework

| Technology | Note |
|-----------|------|
| Kotlin 2.4 | Primary language |
| Java 21 | Toolchain |
| Spring Boot 4.1 | Web, Data JPA, Security, Data Redis, Mustache (admin) |
| Gradle (Kotlin DSL) | Build system |

---

## Serialization Compatibility

Application HTTP and Redis serialization use Jackson 3 after the Spring Boot 4 migration. Jackson 2 remains isolated to libraries that require it for JSONB mapping and JWT handling. Enum values retain their `name()` representation to preserve the existing API and Redis session format. Compatibility tests cover both boundaries.

---

## Database

| Technology | Role |
|-----------|------|
| PostgreSQL | Primary data store (schema-per-tenant) |
| Hibernate (JPA) | ORM — validate mode (schema managed by Flyway) |
| Flyway | Versioned schema migration with multi-schema support |
| hypersistence-utils | JSONB column type mapping for complex data types |

### JSONB Usage

List-type and structured fields are stored as PostgreSQL JSONB columns. This avoids join table overhead for simple nested data while keeping it queryable.

### Flyway Strategy

Migrations are organized separately for shared and tenant-specific schemas. Tenant schema migrations are applied to each tenant on startup.

---

## Session & Cache

| Technology | Role |
|-----------|------|
| Spring Data Redis | Session data storage with TTL |
| Redisson | Distributed rate limiting |

Session state lives entirely in Redis — the backend holds no in-memory state and is horizontally scalable. Redisson backs distributed rate limiting at the API edge so per-session abuse can be throttled across instances.

---

## Security

| Technology | Role |
|-----------|------|
| Spring Security | Stateless session, path-based auth rules |
| JJWT | JWT token generation and validation |

- Worker-side API routes attach the backend shared secret; it is never sent to the browser
- Quiz and shared-result requests use session keys without requiring user login
- JWT support exists for authenticated context, including tenant claims; the browser quiz flow does not obtain a JWT for every request
- Distributed rate limiting prevents abuse on a per-session basis

---

## Song Similarity Engine

A weighted similarity engine ranks songs across multiple attribute categories to surface a list of close matches. Answer submission uses it for no-match and exhausted-question outcomes, and matched results include related songs. Shared-result retrieval can rebuild recommendations from the persisted song even after the session expires. Next-question retrieval still reports empty or exhausted flows as errors.

---

## Testing

| Technology | Role |
|-----------|------|
| MockK | Unit tests for service layer |
| Testcontainers | Integration tests with real PostgreSQL + Redis containers |
| JaCoCo | Code coverage enforcement |

---

## Admin UI

The `:admin` module serves a server-rendered management interface (Mustache templates, local only) for managing all domain data including bulk CSV import. Spotify and YouTube integrations help curate content. Pasted album and concert page HTML can be parsed into candidates for review before import. Admin forms retain CSRF protection, and non-local profiles require an admin role.

---

## API Documentation

SpringDoc OpenAPI — JSON specification available in the local profile. Development and production profiles disable both the specification and Swagger UI.
