## Context

The repository currently contains no application code—only the OpenSpec planning tree—so this change establishes the initial runnable service in addition to the behaviors in `proposal.md`. Constraints that shape the approach:

- Greenfield: there is no existing stack, database schema, or deployment to stay compatible with.
- Security requirements are hard acceptance criteria: hashed passwords, server-side-only secrets, server-enforced RBAC and class isolation.
- The service must run under Docker Compose and expose `GET /health`.
- The stack is deliberately course-sized: Flask + SQLite, one runnable service, and no heavyweight infrastructure.

## Goals / Non-Goals

**Goals:**

- A minimal, conventional web app: Flask pages for login and materials, a JSON API, and a SQLite relational store.
- A single enforcement point for authentication, role, and class checks on the server.
- A deployable Compose topology with no secrets committed to the repository.

**Non-Goals:**

- Password reset, email verification, SSO, or account self-registration flows.
- Multi-class membership or a separate admin role; each account belongs to exactly one class.
- Retrieval Q&A (检索问答) over the knowledge base—no semantic search, ranking, embeddings, vector stores, or RAG; the knowledge base here is a queryable store of uploaded records.
- A chat assistant or conversational agent (对话助手).
- Homework submission or grading workflows (作业提交与批改).
- Production high availability (生产高可用)—multi-instance scaling, load balancing, or failover; this change targets a single-instance course deployment.
- Streaming/previewing uploaded file contents in the browser.

## Decisions

1. **Stack: Python (Flask) + SQLite.**
   Rationale: a course-appropriate, minimal stack with a single runtime for pages and API; Flask's request/session/multipart machinery is mature, and SQLite provides real relational persistence (foreign keys, `CHECK`, `UNIQUE`) without a separate database service. Alternatives considered: Node.js/Express + PostgreSQL (viable, but adds a second service and an npm toolchain with no benefit at this scale); in-memory storage (unacceptable—persistence is a requirement).

2. **Auth: server-side sessions with a signed, HttpOnly session cookie.**
   `POST /api/auth/login` verifies credentials and creates a server-side session record (Flask-Session with a server-side store); the browser holds only an opaque session ID in a signed `HttpOnly` cookie. Protected pages redirect to `/login` when the session is absent or invalid; protected API routes return `401`. Rationale: sessions are revocable via logout, no client-side secret is stored, and this matches the redirect requirement. Alternative considered: stateless JWT (harder to revoke; no requirement for statelessness).

3. **Password storage: bcrypt with a per-password salt.**
   Only `bcrypt(password)` is ever stored or compared; plaintext never touches persistence or logs. Alternative considered: Argon2id (slightly stronger, less ubiquitous; bcrypt is acceptable for this scope).

4. **Roles: `role` column on `users` constrained to `teacher` or `student`, checked by server middleware.**
   The upload route runs a `require_role("teacher")` check before any handler logic, so students get `403` and no record is written, while unauthenticated requests get `401`. The frontend may hide controls for UX, but every privileged route independently re-checks the role.

5. **Class isolation: `class_id` on `users` and `materials`, with server-side filtering on every material read.**
   The session user's `class_id` is the only class value the server trusts. `GET /api/classes/:classId/materials` compares `:classId` to the session user's class and returns `403` on mismatch; `GET /api/materials/:id` resolves the material's owning class and applies the same comparison. List queries always include `WHERE class_id = ?` bound to the server-derived class; no client-supplied value is ever used in a query. This is enforced in route-level checks, not in the UI.

6. **Knowledge base: files on a Docker volume, metadata rows in SQLite.**
   Upload data flow:
   1. The browser sends a multipart `POST /api/materials` with the session cookie.
   2. Session middleware: missing or invalid session → `401`.
   3. `require_role("teacher")`: student → `403`; no write is attempted.
   4. Validate the size cap and MIME allowlist; reject oversized or disallowed uploads.
   5. Generate a server-side `storage_key` (UUID-derived, never the client filename) to prevent path traversal.
   6. Save the file under the uploads volume, outside the web root.
   7. Insert `materials(class_id, uploaded_by, original_filename, storage_key, mime_type, size_bytes, created_at)` using the session user's class and user id.
   8. Respond `201`; if the insert fails, delete the orphaned file.
   `GET /api/classes/:classId/materials` is the class-scoped query over these rows, so a newly uploaded record is immediately returned by a follow-up list request.

7. **Secrets: environment variables only, injected via Compose.**
   `SESSION_SECRET` and the database file path are read from the server environment; `docker-compose.yml` references an untracked `.env` file and the repository ships only a `.env.example` with placeholders. No secret literal is committed.

8. **Compose and health.**
   `docker-compose.yml` defines a single `web` service: an image built from `Dockerfile`, mounted volumes for the SQLite database and uploads, `env_file: .env`, a port mapping, and a container healthcheck that requests `GET /health`. The app answers `200` on `/health` once it has started and the database is reachable, making `docker compose up` plus the healthcheck the smoke-testable deployment contract.

9. **Data model.**

   - `classes(id, name, created_at)`
   - `users(id, username UNIQUE, password_hash, role CHECK IN ('teacher','student'), class_id FK, created_at)`
   - `materials(id, class_id FK, uploaded_by FK, original_filename, storage_key, mime_type, size_bytes, created_at)`

## Risks / Trade-offs

- [Session hijacking via cookie theft] → set `HttpOnly`, `SameSite=Lax`, and `Secure` in production; keep sessions server-side with an idle timeout.
- [CSRF on state-changing endpoints] → `SameSite=Lax` plus a per-session CSRF token checked on uploads and logout.
- [Cross-class IDOR] → every read route resolves the target class or material server-side and compares it to the session user's class before returning data; tests call class B endpoints with a class A session.
- [Upload abuse (oversized or malicious files)] → size cap and MIME allowlist, files stored outside the web root under server-generated keys.
- [Login enumeration / brute force] → uniform error message for unknown user vs. wrong password; optional rate limiting on the login endpoint.
- [bcrypt cost regressions] → centralize the cost factor; existing hashes remain verifiable even if the factor is raised later.
- [Secret leakage] → `.env` gitignored; CI/review checks reject committed secret patterns; only `.env.example` is tracked.
- [SQLite write contention] → the access pattern is a single writer at course scale; enable WAL mode, and PostgreSQL remains a drop-in upgrade path if load ever requires it.

## Migration Plan

1. `init_db` creates the schema idempotently at app startup (greenfield—no data to migrate).
2. Seed one demo class with one teacher and one student (bcrypt-hashed passwords) for verification.
3. Rollout is a fresh `docker compose up`; rollback is re-deploying the previous image, since there is no prior production data.

## Open Questions

None—all decisions that affect the specs, approach, or task breakdown are resolved above.