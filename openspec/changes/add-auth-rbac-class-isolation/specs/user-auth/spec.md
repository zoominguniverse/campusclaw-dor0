## Purpose

Establishes account/password authentication for teachers and students so that protected application areas are only reachable by signed-in users.

## ADDED Requirements

### Requirement: Login with account and password

The system SHALL allow both teachers and students to sign in with an account identifier and a password.

#### Scenario: Successful login

- **WHEN** a teacher or student submits a valid account identifier and password
- **THEN** the system authenticates them, creates an authenticated session, and lets them reach protected pages

#### Scenario: Invalid credentials

- **WHEN** a user submits a wrong account identifier or password
- **THEN** the system rejects the login with an error, does not create an authenticated session, and keeps the user on the login page

### Requirement: Protected pages require authentication

The system SHALL prevent unauthenticated users from viewing protected pages.

#### Scenario: Anonymous access to a protected page

- **WHEN** an unauthenticated user requests a protected page
- **THEN** the system redirects them to the login page instead of rendering the protected page

### Requirement: Passwords are never stored in plaintext

The system SHALL store user passwords only as salted hashes and SHALL NEVER persist or expose plaintext passwords.

#### Scenario: Password storage

- **WHEN** a user account password is created or changed
- **THEN** the system stores only a salted hash of the password and no plaintext copy anywhere

#### Scenario: Password verification

- **WHEN** a user signs in with a password
- **THEN** the system verifies the submitted password against the stored salted hash rather than comparing plaintext values