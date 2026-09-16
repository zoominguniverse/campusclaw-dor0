## 1. Scaffold and deployment baseline

- [ ] 1.1 Initialize the Node.js/Express application (package.json, entrypoint, Dockerfile) and verify `docker compose up` starts the app with `GET /health` returning 200
- [ ] 1.2 Add `docker-compose.yml` with the app and PostgreSQL services, a healthcheck, an uploads volume, `.env.example`, and `.gitignore` entries for `.env` and uploads; verify both services start cleanly and no secret literal is tracked by git

## 2. Data layer and configuration

- [ ] 2.1 Add the schema migration/init for `classes`, `users` (unique username, bcrypt `password_hash`, `role` CHECK teacher/student, `class_id` FK), and `materials` (class FK, uploader FK, metadata columns); verify migration runs at startup and all tables/constraints exist
- [ ] 2.2 Implement a config module that reads `DATABASE_URL` and `SESSION_SECRET` from server environment variables and fails loudly when missing; verify startup errors without them and succeeds with them set
- [ ] 2.3 Add a seed script creating one teacher and one student in a demo class with bcrypt-hashed passwords; verify the database contains only salted hashes and no plaintext password values

## 3. Authentication

- [ ] 3.1 Implement `POST /api/auth/login` that verifies bcrypt hashes and creates a server-side session; verify with tests: valid teacher and student credentials return success with a session cookie, wrong password and unknown username are both rejected with a uniform error and no session
- [ ] 3.2 Add session middleware and make protected pages redirect to `/login`; verify an unauthenticated browser request to a protected page receives a redirect to `/login` and a valid session renders the page
- [ ] 3.3 Implement logout that destroys the session; verify the session is invalidated and a subsequent protected-page request redirects to `/login`

## 4. Role-based permissions

- [ ] 4.1 Add `requireRole('teacher')` middleware in front of the upload handler; verify tests: teacher upload proceeds, student upload returns 403 and writes no `materials` row, anonymous upload returns 401
- [ ] 4.2 Verify server-side role checks hold when the endpoint is called directly without the frontend (e.g., raw HTTP client); student request must still be rejected

## 5. Class isolation

- [ ] 5.1 Implement `GET /api/classes/:classId/materials` with a server-side check comparing `:classId` to the session user's class; verify tests: own class returns 200 with the class's records, a class A member requesting class B returns 403 with no class B data
- [ ] 5.2 Implement `GET /api/materials/:id` with ownership validation; verify a class A member requesting a class B material returns 403 and receives no material data
- [ ] 5.3 Add a test that issues cross-class requests directly (no frontend controls involved) and verify rejection still occurs

## 6. Knowledge-base upload and listing

- [ ] 6.1 Implement multipart `POST /api/materials` that stores the file on the uploads volume under a server-generated key and writes the metadata row; verify a teacher upload returns success, persists the file, and records `class_id`/`uploaded_by` correctly
- [ ] 6.2 Implement the class-scoped material list query; verify a newly uploaded material appears in the teacher's own class list and the response contains only that class's materials

## 7. Security hardening

- [ ] 7.1 Set `HttpOnly` and `SameSite` cookie attributes, add CSRF protection on state-changing routes, cap upload size, and restrict MIME types; verify tests cover cookie flags, oversized upload rejection, and CSRF rejection
- [ ] 7.2 Add a secret-leak check (grep/CI step) confirming no secret literal appears in committed files; verify it passes on the committed tree

## 8. Acceptance and documentation

- [ ] 8.1 Add an end-to-end acceptance test covering every user criterion: teacher and student login, redirect of anonymous protected access, student upload rejection, class A vs. class B material rejection, teacher upload visible in the class material list; verify all pass
- [ ] 8.2 Perform a full `docker compose up` smoke test hitting `GET /health` and document startup and demo credentials in the README; verify a clean clone can start and pass health
- [ ] 8.3 Run `openspec validate --strict` on this change and verify it reports no errors