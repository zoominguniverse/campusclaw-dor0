## Why

The campus app has no authentication, role enforcement, or data boundaries yet, so anyone could reach protected pages, students could tamper with the knowledge base, and one class's materials would be visible to every other class. This change adds the security foundation—login, teacher/student roles, and class-scoped data isolation—and the first teacher workflow on top of it: uploading teaching materials into the knowledge base.

## What Changes

- Introduce account/password login for both teachers and students; unauthenticated requests to protected pages are redirected to the login page.
- Store user passwords only as salted hashes—never plaintext—and keep application secrets exclusively in server-side environment variables.
- Introduce teacher and student roles and enforce permissions on the server: students are rejected from the material-upload API regardless of frontend UI.
- Make class membership a server-enforced data boundary: requests from a member of class A for class B materials are rejected, independent of any frontend button visibility.
- Add a teacher-only endpoint that writes an uploaded teaching material into the knowledge base, and a per-class material listing endpoint that returns the newly uploaded record.
- Add a `GET /health` endpoint and a Docker Compose configuration so the whole service can be started with `docker compose up`.
- **BREAKING**: the repository currently contains no application code, so this change also establishes the minimal runnable service (web app, database, and Compose topology) that these behaviors operate on.

## Non-goals

Out of scope for this course and not built or judged by this change:

- **Retrieval Q&A (检索问答)**: no semantic search, ranking, embeddings, vector stores, or RAG over the knowledge base; it stays a class-scoped record store with a list endpoint.
- **Chat assistant (对话助手)**: no chatbot or conversational agent.
- **Homework submission & grading (作业提交与批改)**: no assignment hand-in, grading, or feedback workflows.
- **SSO / unified identity (SSO/统一身份认证)**: no OAuth, OIDC, or enterprise identity integration; login is only against pre-provisioned accounts.
- **Production HA (生产高可用)**: no multi-instance scaling, load balancing, or failover; the target is a single-instance course deployment.
- **Account self-service**: no self-registration, email verification, or password reset flows.

## Capabilities

### New Capabilities
- `user-auth`: account/password login for teachers and students, protected-page redirects, and hashed credential storage.
- `rbac`: role assignment (teacher/student) and server-side permission enforcement.
- `class-isolation`: class membership as the server-enforced data boundary for materials.
- `knowledge-base`: teacher upload of teaching materials into the knowledge base and class-scoped material listing.
- `service-runtime`: operational contract—`GET /health`, secret handling via server-side environment variables, and Docker Compose startup.

### Modified Capabilities
<!-- None: this is the first capability set in the repository. -->

## Impact

- **New application surface**: a runnable web service with pages for login and material listing, a database for users/memberships/materials, and session-based authentication.
- **APIs**: new endpoints for login, logout, material upload (teacher-only), and class-scoped material listing; plus `GET /health`.
- **Dependencies**: password-hashing library, server-side session management, database driver/ORM, and a Docker Compose environment.
- **Deployment**: `docker-compose.yml` and environment-variable-driven configuration for secrets (DB credentials, session secret).