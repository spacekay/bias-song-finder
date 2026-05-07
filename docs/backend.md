[🇰🇷 한국어](ko/backend.md)

# Backend Tech Stack

## Core Framework

| Technology | Note |
|-----------|------|
| Kotlin | Primary language |
| Java 21 | Toolchain |
| Spring Boot 3.4 | Web, Data JPA, Security, Data Redis, Mustache (admin) |
| Gradle (Kotlin DSL) | Build system |

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

- Frontend authenticates via a shared secret to obtain a short-lived JWT
- Quiz endpoints use session key as the identifier and are exempt from JWT auth
- Distributed rate limiting prevents abuse on a per-session basis

---

## Song Similarity Engine

A weighted similarity engine ranks songs across multiple attribute categories to surface a list of close matches. It is invoked whenever the quiz cannot land on a single song — when the candidate pool is empty, when all available questions have been exhausted, or when the user reaches the no-match path — so the response always carries a meaningful set of suggestions instead of an error.

---

## Testing

| Technology | Role |
|-----------|------|
| MockK | Unit tests for service layer |
| Testcontainers | Integration tests with real PostgreSQL + Redis containers |
| JaCoCo | Code coverage enforcement |

---

## Admin UI

The `:admin` module serves a server-rendered management interface (Mustache templates, local only) for managing all domain data including bulk CSV import. Spotify and YouTube integrations are wired into the admin flow to streamline content curation.

---

## API Documentation

SpringDoc OpenAPI — JSON spec available for API consumers.
