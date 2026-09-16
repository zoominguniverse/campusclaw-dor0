## Context

The repository currently contains no application code—only the OpenSpec planning tree—so this change establishes the initial runnable service in addition to the behaviors in `proposal.md`. Constraints that shape the approach:

- Greenfield: there is no existing stack, database schema, or deployment to stay compatible with.
- Security requirements are hard acceptance criteria: hashed passwords, server-side-only secrets, server-enforced RBAC and class isolation.
- The service must run under Docker Compose and expose `GET /health`.

## Goals / Non-Goals

**Goals:**

- A minimal, conventional web app: browser pages for login and materials, a JSON/HTML API behind them, and a relational store.
- Single enforcement point for authentication, role, and class checks on the server.
- A deployable Compose topology with no secrets committed to the repository.

**Non-Goals:**

- Password reset, email verification, SSO, or account self-registration flows.
- Multi-class membership or a separate admin role; each account belongs to exactly one class.
- Semantic/vector knowledge-base features (embeddings, search ranking, RAG); "knowledge base" here means a queryable, persistent store of uploaded material records.
- Streaming/previewing uploaded file contents in the browser.

## Decisions

1. **Stack: Node.js (LTS) + Express + PostgreSQL.**
   Rationale: minimal and widely understood, single runtime for pages and API, mature session/multipart ecosystem, and a real database that fits the Compose multi-service requirement. Alternatives considered: Python/FastAPI (equally viable, no advantage here); SQLite (simpler but weakens the Compose topology and concurrent-write story).

2. **Auth: server-side sessions with a signed, HttpOnly cookie.**
   The login endpoint sets a session cookie; protected pages redirect to `/login` when it is absent or invalid; protected API routes return `401`. Rationale: trivially revocable, no client-side secret storage, and matches the redirect requirement. Alternative considered: stateless JWT (harder to revoke; no requirement for statelessness).

3. **Password storage: bcrypt with a per-password salt.**
   Only `bcrypt(password)` is ever stored or compared; plaintext never touches persistence or logs. Alternative considered: Argon2id (slightly stronger, less ubiquitous; bcrypt is acceptable for this scope).

4. **Roles: `role` column on `users` constrained to `teacher` or `student`, checked by server middleware.**
   The upload route runs a `requireRole('teacher')` middleware before any handler logic, so students get `403` and no record is written. The frontend may hide controls for UX, but every privileged route independently re-checks the role.

5. **Class isolation: `class_id` on `users`, `class_id` on `materials`, and server-side ownership checks on every material read.**
   `GET /api/classes/:classId/materials` and `GET /api/materials/:id` compare the requested class/material against the session user's `class_id` and return `403` on mismatch. Queries are always scoped by the server-derived class; no client-supplied value is trusted. This is enforced in route middleware, not in the UI.

6. **Knowledge base: uploaded files stored on a Docker volume, with a metadata row in PostgreSQL.**
   `POST /api/materials` (multipart) writes `materials(id, class_id, uploaded_by, original_filename, storage_key, mime_type, size, created_at)`; `storage_key` is server-generated (never the client filename) to prevent path traversal. The material list is a class-scoped query over these rows, so a newly uploaded record is immediately returned.

7. **Secrets: environment variables only, injected via Compose.**
   `SESSION_SECRET` and `DATABASE_URL`/`POSTGRES_PASSWORD` are read from server environment; `docker-compose.yml` references an untracked `.env` file and ships only a `.env.example` with placeholders. No secret literal is committed.

8. **Data model.**

   - `classes(id, name, created_at)`
   - `users(id, username UNIQUE, password_hash, role CHECK IN ('teacher','student'), class_id FK, created_at)`
   - `materials(id, class_id FK, uploaded_by FK, original_filename, storage_key, mime_type, size_bytes, created_at)`

## Risks / Trade-offs

- [Session hijacking via cookie theft] → set `HttpOnly`, `SameSite=Lax`, and `Secure` in production; keep sessions server-side and short-lived with idle timeout.
- [CSRF on state-changing endpoints] → `SameSite=Lax` plus a per-session CSRF token checked on uploads/logout.
- [Cross-class IDOR] → every read route resolves the target class/material server-side and compares to the session user's class before returning data; add tests that call class B endpoints with a class A token.
- [Upload abuse (oversized or malicious files)] → size cap and MIME allowlist, files stored outside the web root under server-generated keys.
- [Login enumeration / brute force] → uniform error message for unknown user vs. wrong password; optional rate limiting on the login endpoint.
- [bcrypt cost regressions] → centralize the cost factor; existing hashes remain verifiable even if the factor is raised later.
- [Secret leakage] → `.env` gitignored; CI/review checks reject committed secret patterns; only `.env.example` is tracked.

## Migration Plan

1. Build the schema with an idempotent migration/init step that runs at app startup (greenfield—no data to migrate).
2. Seed one teacher and one student per demo class for verification.
3. Rollout is a fresh `docker compose up`; rollback is re-deploying the previous image, since there is no prior production data.

## Open Questions

None—all decisions that affect the specs, approach, or task breakdown are resolved above.