## Purpose

Defines teacher and student roles and enforces role-based permissions on the server so that restricted actions cannot be bypassed from the frontend.

## ADDED Requirements

### Requirement: Every account has a role

The system SHALL assign every authenticated account exactly one role: teacher or student.

#### Scenario: Role is bound to an account

- **WHEN** an authenticated account performs an action
- **THEN** the system evaluates permissions using that account's assigned teacher or student role

### Requirement: Students cannot upload teaching materials

The system SHALL reject material uploads from student accounts with an HTTP 403 Forbidden response and SHALL NOT create any material record.

#### Scenario: Student attempts to upload

- **WHEN** a student account calls the material upload endpoint
- **THEN** the system responds with HTTP 403 Forbidden and creates no material record

#### Scenario: Teacher uploads material

- **WHEN** a teacher account calls the material upload endpoint with valid input
- **THEN** the system accepts the request and processes the upload

### Requirement: Permissions are enforced on the server

The system SHALL evaluate role permissions on the server for every protected request, independent of what the frontend shows or hides.

#### Scenario: Bypassed frontend controls

- **WHEN** a protected request is issued directly, without relying on any frontend button or navigation control
- **THEN** the server still applies the role check and rejects requests the account's role is not permitted to make