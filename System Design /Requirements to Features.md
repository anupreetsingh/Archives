# Requirements to Features

When designing a system, separate the discussion into two main parts:

1. **Requirements** - describe what the system needs to do or how the system needs to behave.
2. **Features** - describe the concrete capabilities that implement those requirements.

## 1. Requirements

Requirements define the expected behavior, qualities, and constraints of the system.

Requirements can be divided into two categories:

1. Functional requirements
2. Non-functional requirements

### Functional Requirements

Functional requirements describe what the system must do from a user or business perspective.

They usually describe visible behavior, workflows, or actions.

They answer questions like:

- What should users be able to do?
- What workflows should the system support?
- What business capabilities must exist?

Examples:

- The system must provide user authentication.
- Users must be able to search for products.
- Users must be able to place an order.
- Admins must be able to manage inventory.

### Non-Functional Requirements

Non-functional requirements describe how the system must behave.

They usually describe system qualities, constraints, and operating expectations.

They answer questions like:

- How fast should the system be?
- How reliable should the system be?
- How secure should the system be?
- How scalable should the system be?
- How easy should the system be to operate and maintain?

Common categories:

- Performance
- Scalability
- Availability
- Reliability
- Security
- Usability
- Maintainability
- Observability
- Compliance

Examples:

- The system must handle 10,000 queries per minute.
- The system must maintain 99.9% availability.
- User passwords must be stored securely.
- Search results must return within 200 ms for most requests.

## 2. Features

Features are concrete product or technical capabilities that implement requirements.

A feature can implement:

- A functional requirement
- A non-functional requirement
- Both a functional and non-functional requirement

Features can be user-facing or internal.

Examples of user-facing features:

- Login page
- Registration flow
- Search page
- Checkout flow
- Admin dashboard

Examples of internal or technical features:

- Caching
- Load balancing
- Rate limiting
- Database indexing
- Read replicas
- Monitoring and alerting

### Mapping Requirements to Features

Within the features discussion, map each feature back to the requirement it supports.

This helps show:

- Why the feature exists.
- Which requirement the feature implements.
- Whether a requirement is fully supported by the proposed design.
- Whether a feature is unnecessary because it does not support any requirement.

### Requirement to Feature Map

| Requirement | Type | Features | Success Criteria |
| --- | --- | --- | --- |
| The system must provide user authentication. | Functional | Login page<br>Registration flow<br>Password reset flow<br>Session management<br>Logout | Users can create an account, log in, stay authenticated across requests, and log out. |
| The system must handle 10,000 queries per minute. | Non-functional | Load balancing<br>Search result caching<br>Database read replicas<br>Search indexing<br>Horizontal scaling | The system can sustain 10,000 queries per minute while meeting latency and error-rate targets. |

### Feature to Requirement Map

| Feature | Requirement Implemented | Requirement Type | Notes |
| --- | --- | --- | --- |
| Login page | The system must provide user authentication. | Functional | Gives users a way to access their account. |
| Password reset flow | The system must provide user authentication. | Functional | Supports account recovery. |
| Session management | The system must provide user authentication. | Functional | Keeps users authenticated across requests. |
| Load balancing | The system must handle 10,000 queries per minute. | Non-functional | Distributes traffic across servers. |
| Search result caching | The system must handle 10,000 queries per minute. | Non-functional | Reduces repeated backend work. |
| Database read replicas | The system must handle 10,000 queries per minute. | Non-functional | Increases read capacity. |
