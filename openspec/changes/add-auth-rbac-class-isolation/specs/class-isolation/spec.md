## Purpose

Makes class membership a server-enforced data boundary so that members of one class can never read another class's materials.

## ADDED Requirements

### Requirement: Class-scoped access to materials

The system SHALL scope every material read to the requesting user's class and SHALL reject access to materials belonging to a different class.

#### Scenario: Cross-class material list

- **WHEN** a signed-in member of class A requests the material list of class B
- **THEN** the system rejects the request with a forbidden response and returns no class B materials

#### Scenario: Cross-class single material

- **WHEN** a signed-in member of class A requests a single material owned by class B
- **THEN** the system rejects the request with a forbidden response and returns no material data

### Requirement: Class isolation is enforced on the server

The system SHALL verify class membership on the server for every material request, regardless of frontend filtering or hidden controls.

#### Scenario: Direct request outside the frontend

- **WHEN** a signed-in member of class A issues a material request for class B directly, without using the frontend
- **THEN** the server rejects the request even though no frontend control exposed it