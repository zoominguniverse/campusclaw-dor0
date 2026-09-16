## Purpose

Defines the operational contract for running and deploying the service, including a health check and secret handling.

## ADDED Requirements

### Requirement: Health endpoint

The system SHALL expose a `GET /health` endpoint that reports service liveness.

#### Scenario: Healthy service

- **WHEN** the service is running and an HTTP request is made to `GET /health`
- **THEN** the system responds with a success status indicating the service is healthy

### Requirement: Docker Compose startup

The system SHALL be startable with Docker Compose, including all required dependencies.

#### Scenario: Starting the stack

- **WHEN** an operator runs `docker compose up` from the project root
- **THEN** the service and its dependencies start and the `GET /health` endpoint responds

### Requirement: Secrets live only in server-side environment variables

The system SHALL obtain all secrets—such as session signing keys and database credentials—from server-side environment variables, and SHALL NOT hardcode them in source or ship them to clients.

#### Scenario: Secrets are resolved from the server environment

- **WHEN** the service reads configuration for a secret value
- **THEN** the value comes from a server-side environment variable and is never embedded in source code or delivered to a browser