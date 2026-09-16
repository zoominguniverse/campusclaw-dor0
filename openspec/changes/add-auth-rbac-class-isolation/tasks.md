## 1. Scaffold and deployment baseline

- [ ] 1.1 Initialize the Flask application (app entrypoint, `requirements.txt`, `Dockerfile`) exposing `GET /health`; verify `docker compose up` builds the image and `/health` returns 200
- [ ] 1.2 Add `docker-compose.yml` with the `web` service, mounted volumes for the SQLite database and uploads, a container healthcheck, and `env_file`, plus `.env.example` and `.gitignore` entries for `.env`, the database file, and uploads; verify the stack starts cleanly and git tracks no secret literal

## 2. Data layer, configuration, and seed data

- [ ] 2.1 Add idempotent schema init for `classes`, `users` (unique username, bcrypt `password_hash`, `role` CHECK teacher/student, `class_id` FK), and `materials` (class FK, uploader FK, metadata columns); verify init runs at startup and all tables and constraints exist
- [ ] 2.2 Implement a config module that reads `DATABASE_PATH` (or `DATABASE_URL`) and `SESSION_SECRET` from server environment variables and fails loudly when missing; verify startup errors without them and succeeds with them set
- [ ] 2.3 Add a seed script creating one demo class plus one teacher and one student in it, with bcrypt-hashed passwords; verify the database contains only salted hashes and no plaintext password values

## 3. Authentication

- [ ] 3.1 Implement `POST /api/auth/login` that verifies bcrypt hashes and creates a server-side session; verify tests: valid teacher and student credentials return success with a session cookie, and wrong password and unknown username are both rejected with a uniform error and no session
- [ ] 3.2 Add session middleware and make protected pages redirect to `/login`; verify an unauthenticated browser request to a protected page receives a redirect to `/login` and a valid session renders the page
- [ ] 3.3 Implement logout that destroys the session; verify the session is invalidated and a subsequent protected-page request redirects to `/login`

## 4. Role-based permissions

- [ ] 4.1 Add a `require_role("teacher")` check in front of the upload handler; verify tests: teacher upload proceeds, student upload returns 403 and writes no `materials` row, and anonymous upload returns 401
- [ ] 4.2 Verify server-side role checks hold when the endpoint is called directly without the frontend (raw HTTP client); verify the student request is still rejected with 403

## 5. Class isolation

- [ ] 5.1 Implement `GET /api/classes/:classId/materials` with a server-side check comparing `:classId` to the session user's class; verify tests: own class returns 200 with its records, and a class A member requesting class B returns 403 with no class B data
- [ ] 5.2 Implement `GET /api/materials/:id` with ownership validation; verify a class A member requesting a class B material returns 403 and receives no material data
- [ ] 5.3 Add a test that issues cross-class requests directly (no frontend controls involved); verify rejection still occurs

## 6. Knowledge-base upload and listing

- [ ] 6.1 Implement multipart `POST /api/materials` that stores the file on the uploads volume under a server-generated key and writes the metadata row; verify a teacher upload returns 201, persists the file, and records `class_id` and `uploaded_by` correctly
- [ ] 6.2 Implement the class-scoped material list query; verify a newly uploaded material appears in the teacher's own class list and the response contains only that class's materials

## 7. Security hardening

- [ ] 7.1 Set `HttpOnly` and `SameSite` cookie attributes, add CSRF protection on state-changing routes, cap upload size, and restrict MIME types; verify tests cover cookie flags, oversized upload rejection, and CSRF rejection
- [ ] 7.2 Add a secret-leak check (grep/CI step) confirming no secret literal appears in committed files; verify it passes on the committed tree

## 8. Acceptance, validation, and documentation

- [ ] 8.1 Add an end-to-end acceptance test covering every user criterion: teacher and student login, anonymous redirect, student upload 403, class A vs. class B material 403, and teacher upload visible in the class material list; verify all pass
- [ ] 8.2 Perform a full `docker compose up` smoke test hitting `GET /health`, and document startup plus demo credentials in the README; verify a clean clone starts and passes the health check
- [ ] 8.3 Run `openspec validate --strict` on this change; verify it reports no errors