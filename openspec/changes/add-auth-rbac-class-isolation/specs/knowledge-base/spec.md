## Purpose

Lets teachers add teaching materials to the knowledge base and lets each class retrieve the materials that belong to it.

## ADDED Requirements

### Requirement: Teacher upload writes to the knowledge base

The system SHALL write a teacher-uploaded teaching material into the knowledge base as a persistent, class-owned record.

#### Scenario: Successful teacher upload

- **WHEN** a teacher uploads a valid teaching material for their class
- **THEN** the system persists the material in the knowledge base together with its owning class and uploader, and responds with a success status

#### Scenario: Student upload is forbidden

- **WHEN** a student account calls the material upload endpoint
- **THEN** the system responds with HTTP 403 Forbidden and writes no record

#### Scenario: Unauthenticated upload

- **WHEN** an unauthenticated request calls the material upload endpoint
- **THEN** the system rejects the request without writing any record

### Requirement: Class material list includes uploaded records

The system SHALL return the requesting user's class materials, including records created via teacher upload.

#### Scenario: Newly uploaded material appears in list

- **WHEN** a signed-in member of a class requests that class's material list after a teacher has uploaded a material for it
- **THEN** the response includes the uploaded material record

#### Scenario: List contains only own class materials

- **WHEN** a signed-in member of a class requests the material list
- **THEN** the response contains materials of that class only and no materials from other classes