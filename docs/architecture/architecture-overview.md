# M0 — Architecture Overview

## 1. Purpose

This document provides the consolidated architectural definition of the **Biometric Security Lab**.

It represents the final architectural documentation produced during **Milestone 0 — Project Definition & Threat Model** and serves as the central reference for understanding the system before implementation and security testing begin.

The purpose of this document is to explain:

- what the laboratory is designed to demonstrate,
- which components form the system,
- how those components interact,
- where security controls are enforced,
- where trust boundaries exist,
- which assets require protection,
- what the primary attack surface is,
- how security telemetry flows through the system,
- and which assumptions define the validity of the laboratory.

This document complements the individual architecture and security documents contained in:

```text
docs/
├── architecture/
│   ├── system-architecture.drawio.png
│   ├── logical-architecture.drawio.png
│   ├── docker-topology.drawio.png
│   ├── service-boundary-model.drawio.png
│   ├── trust-boundaries.drawio.png
│   └── attack-surface-map.drawio.png
│
└── security/
    ├── assets.md
    ├── data-flow.md
    ├── security-assumptions.md
    └── threat-model.md
```

The architecture defined here is intentionally designed as a **security laboratory**, rather than as a production deployment.

The objective is not only to make the application functional, but to create an environment in which security controls can be attacked, observed, detected, investigated, remediated, and retested.

---

# 2. Project Context

The Biometric Security Lab is a self-hosted cybersecurity laboratory centered around a biometric access-control application.

The system combines:

- a web application,
- a dedicated backend/API,
- relational data storage,
- biometric recognition,
- containerized infrastructure,
- security monitoring,
- controlled attack infrastructure,
- and SOC-oriented investigation.

The project therefore combines both **application security** and **defensive security**.

The intended security validation cycle is:

```text
Architecture
     ↓
Security Controls
     ↓
Controlled Attack
     ↓
Application Behavior
     ↓
Security Telemetry
     ↓
Detection
     ↓
SOC Investigation
     ↓
Remediation
     ↓
Retest
```

This means that an attack is not considered the final objective of the laboratory.

The attack is instead used as a mechanism for validating whether the implemented security controls and monitoring capabilities behave as expected.

---

# 3. Architectural Objectives

The architecture is designed around the following objectives.

## 3.1 Application Security

The application must provide a dedicated security boundary between the client and internal services.

Security-sensitive decisions must therefore be performed by the backend rather than being trusted to the frontend.

The backend is responsible for:

- authentication,
- authorization,
- business logic,
- API security,
- input validation,
- rate limiting,
- and security telemetry.

---

## 3.2 Biometric Security

The system incorporates facial recognition as part of the identity and authentication flow.

The biometric subsystem is treated as a security-sensitive component because biometric information can affect authentication and access-control decisions.

The architecture therefore separates biometric processing from the main application logic.

CompreFace is used as the dedicated biometric recognition service.

The application communicates with the biometric subsystem through the backend rather than exposing the biometric service directly to the client.

---

## 3.3 Defensive Monitoring

Security telemetry is treated as a first-class capability of the laboratory.

Application activity should produce security-relevant events that can be collected and analyzed by Wazuh.

The intended defensive pipeline is:

```text
Application Activity
        ↓
Security Event
        ↓
Structured Log
        ↓
Wazuh
        ↓
Decoder
        ↓
Detection Rule
        ↓
Alert
        ↓
SOC Investigation
```

This allows the project to demonstrate not only whether an attack succeeds, but also whether the attack can be detected and investigated.

---

## 3.4 Controlled Adversarial Testing

The architecture includes a dedicated attack environment.

Parrot OS is used as the primary controlled attacker environment.

The attacker is expected to interact with the exposed application surface and generate controlled malicious or suspicious activity.

The attacker does not receive direct access to protected internal services unless a specific laboratory scenario is intentionally designed to test such access.

---

# 4. High-Level System Architecture

The high-level architecture is represented by the system architecture diagram:

```text
Client / Presentation
        │
        │ HTTPS / API
        ▼
Spring Boot Application
        │
        ├───────────────┐
        │               │
        ▼               ▼
 PostgreSQL          PostgreSQL
 Application Data    Application Data
 Users               Users
 Roles               Roles
 Permissions         Permissions
        │
        │
        └───────────────┐
                        ▼
                    Security
                    Telemetry
                        │
                        ▼
                      Wazuh
```

The web presentation layer is implemented using **Next.js**.

The dedicated backend/API layer is implemented using **Spring Boot**.

The backend communicates with the required internal services, including PostgreSQL and CompreFace.

Wazuh operates as the security monitoring and detection layer.

The complete architecture is documented visually in:

```text
docs/architecture/system-architecture.drawio.png
```

---

# 5. Logical Architecture

The logical architecture separates the system into distinct responsibilities.

## 5.1 Presentation Layer

The presentation layer is implemented using:

```text
Next.js
```

Its responsibilities include:

- rendering the web interface,
- interacting with the backend API,
- presenting authentication and application functionality,
- and displaying application responses to the user.

The frontend is considered **untrusted** from a security perspective.

Client-side validation may improve usability, but it must not be considered an authoritative security control.

A malicious user may modify requests, bypass frontend logic, or interact directly with backend endpoints.

Therefore:

```text
Frontend Validation ≠ Security Boundary
```

The authoritative security decisions are performed by the backend.

---

# 6. Backend and Application Security Boundary

The backend is implemented using:

```text
Spring Boot
```

Spring Boot represents the primary application security boundary.

It is responsible for receiving requests from the presentation layer and applying the security controls required before accessing protected resources.

The backend contains the following conceptual responsibilities:

```text
Authentication
Authorization
Business Logic
API Security
Input Validation
Rate Limiting
Security Telemetry
```

The intended request flow is:

```text
Client
  ↓
Next.js
  ↓
Spring Boot API
  ↓
Security Controls
  ↓
Business Logic
  ↓
Internal Services
```

The backend therefore acts as the enforcement point between an untrusted client and protected resources.

---

# 7. Authentication and Authorization

Authentication and authorization are treated as separate security concepts.

Authentication establishes the identity associated with a request.

Authorization determines what that identity is permitted to perform.

The conceptual flow is:

```text
Authentication
      ↓
Identity
      ↓
Authorization
      ↓
Allowed / Denied Operation
```

Client-provided identity information is not considered trustworthy by itself.

The backend must therefore validate authentication state before allowing access to protected functionality.

Authorization decisions must be performed server-side.

The intended security principle is:

```text
Client
  ↓
Authentication
  ↓
Identity
  ↓
Authorization
  ↓
Protected Resource
```

Biometric recognition may contribute to establishing identity, but biometric recognition does not independently define authorization privileges.

---

# 8. Biometric Recognition Architecture

The laboratory incorporates facial recognition through:

```text
CompreFace
```

CompreFace is treated as a dedicated biometric recognition service.

The conceptual flow is:

```text
Client
  ↓
Next.js
  ↓
Spring Boot
  ↓
CompreFace
  ↓
Biometric Result
  ↓
Authentication / Identity Decision
  ↓
Authorization
```

The frontend does not directly control the biometric authorization decision.

Instead, the backend coordinates the interaction between the application and biometric subsystem.

This design allows the biometric component to remain behind the application's security boundary.

Biometric data is considered a sensitive security asset.

Potential compromise of biometric information may affect:

- user privacy,
- authentication integrity,
- access-control decisions,
- and the overall trustworthiness of the system.

---

# 9. Data Storage

The application uses:

```text
PostgreSQL
```

as its relational database.

PostgreSQL stores application-related information such as:

- users,
- roles,
- permissions,
- and other application data required by the implementation.

Database access is intended to occur through controlled backend operations.

The frontend is not intended to communicate directly with PostgreSQL.

The intended architecture is:

```text
Next.js
   ↓
Spring Boot
   ↓
PostgreSQL
```

rather than:

```text
Next.js
   ↓
PostgreSQL
```

This separation is important because it prevents the frontend from becoming an authoritative database access layer.

---

# 10. Security Monitoring Architecture

Wazuh provides the primary security monitoring and detection capability of the laboratory.

The monitoring architecture is based on the principle that security-relevant application behavior should become observable telemetry.

The intended pipeline is:

```text
Application Activity
        ↓
Security Event
        ↓
Structured Log
        ↓
Wazuh
        ↓
Decoder
        ↓
Detection Rule
        ↓
Alert
        ↓
SOC Analyst
```

Wazuh is therefore not simply an external logging destination.

It represents the defensive security layer of the laboratory.

The purpose of this layer is to determine whether security-relevant behavior can be:

1. generated,
2. collected,
3. interpreted,
4. detected,
5. alerted,
6. and investigated.

---

# 11. SOC Investigation Model

The project is designed to support a SOC-oriented workflow.

When Wazuh generates an alert, the expected process is:

```text
Alert
  ↓
Triage
  ↓
Evidence Collection
  ↓
Timeline
  ↓
Scope
  ↓
Classification
  ↓
MITRE ATT&CK Mapping
  ↓
Response
  ↓
Remediation
  ↓
Retest
```

This separates the laboratory into two complementary perspectives.

## Offensive Perspective

The attacker attempts to:

- discover attack surface,
- manipulate requests,
- abuse authentication,
- bypass authorization,
- exploit application weaknesses,
- or generate other controlled malicious behavior.

## Defensive Perspective

The SOC analyst must:

- recognize the resulting telemetry,
- determine whether the activity is malicious,
- establish scope,
- reconstruct the timeline,
- classify the event,
- identify the relevant technique,
- and recommend remediation.

The same laboratory therefore validates both offensive and defensive capabilities.

---

# 12. Container Architecture

The project uses Docker for application infrastructure.

The container topology separates the main services according to their responsibilities.

The conceptual service structure is:

```text
Application Environment
│
├── Next.js
│
├── Spring Boot
│
├── PostgreSQL
│
├── CompreFace
│
└── Wazuh
```

The exact container configuration may evolve during implementation.

The Docker topology is documented separately in:

```text
docs/architecture/docker-topology.drawio.png
```

The purpose of containerization is to provide:

- reproducibility,
- service isolation,
- controlled configuration,
- easier lifecycle management,
- and a consistent laboratory environment.

Containerization does not automatically establish security.

Each service must still be treated according to its own trust boundary and attack surface.

---

# 13. Service Boundaries

Each major component has a distinct responsibility.

```text
Next.js
Presentation

Spring Boot
Application Security + Business Logic + API

PostgreSQL
Application Data

CompreFace
Biometric Recognition

Wazuh
Security Monitoring + Detection

Parrot
Controlled Attack Infrastructure
```

The service boundary model exists to prevent responsibilities from becoming unnecessarily coupled.

For example:

- the frontend should not become the authorization authority,
- the database should not become directly accessible to the client,
- the biometric engine should not become the application's authorization authority,
- and Wazuh should remain the monitoring/detection layer rather than being embedded entirely inside application logic.

The detailed service boundary model is documented in:

```text
docs/architecture/service-boundary-model.drawio.png
```

---

# 14. Trust Boundaries

The architecture explicitly distinguishes trusted and untrusted components.

The most important trust principle is:

```text
Client = Untrusted
```

The client may be manipulated by an attacker.

Therefore, requests originating from the frontend must be treated as untrusted input.

The backend represents the primary enforcement boundary.

A simplified trust model is:

```text
Untrusted Client
       ↓
========================
Application Trust Boundary
========================
       ↓
Spring Boot
       ↓
Protected Internal Services
```

The internal services are not intended to be directly exposed to untrusted clients.

The detailed trust-boundary model is documented in:

```text
docs/architecture/trust-boundaries.drawio.png
```

The security assumptions associated with these boundaries are documented in:

```text
docs/security/security-assumptions.md
```

---

# 15. Attack Surface

The laboratory intentionally exposes an attack surface for controlled security testing.

The primary attack surface is the application-facing interface.

Conceptually:

```text
Parrot Attacker
       ↓
     HTTP/S
       ↓
Next.js / Application Entry Point
       ↓
Spring Boot API
       ↓
Internal Services
```

Potential attack categories include:

- authentication attacks,
- authorization attacks,
- account enumeration,
- brute-force behavior,
- malformed input,
- API abuse,
- rate-limit bypass attempts,
- session-related attacks,
- biometric authentication abuse,
- and other application-layer scenarios defined during subsequent milestones.

The attack surface must be evaluated according to what is actually exposed by the implementation.

Internal services such as PostgreSQL and CompreFace should not be considered externally accessible attack targets unless the laboratory explicitly introduces a scenario that exposes them.

The attack surface is documented in:

```text
docs/architecture/attack-surface-map.drawio.png
```

---

# 16. Security Assets

The laboratory identifies security-relevant assets that require protection.

The primary asset categories include:

```text
A-01  User Accounts
A-02  Authentication Credentials
A-03  Roles & Permissions
A-04  Biometric Data
```

Additional application, infrastructure, and security-monitoring assets are documented in:

```text
docs/security/assets.md
```

The asset model considers three primary security properties:

```text
Confidentiality
Integrity
Availability
```

Assets are evaluated according to their relevance to these properties and their potential impact if compromised.

Particular attention is given to:

- authentication information,
- authorization information,
- biometric information,
- application data,
- and security telemetry.

Security telemetry is itself treated as a security-critical capability because loss or manipulation of telemetry can reduce the ability of the SOC to detect and investigate attacks.

---

# 17. Data Flow

The architecture defines multiple important data flows.

## 17.1 Application Request Flow

```text
Client
  ↓
Next.js
  ↓
Spring Boot
  ↓
Security Controls
  ↓
Business Logic
  ↓
Internal Service
  ↓
Response
```

---

## 17.2 Database Flow

```text
Spring Boot
     ↓
PostgreSQL
     ↓
Application Data
```

The database is accessed through controlled backend operations.

---

## 17.3 Biometric Flow

```text
Client
  ↓
Next.js
  ↓
Spring Boot
  ↓
CompreFace
  ↓
Biometric Recognition
  ↓
Identity Result
  ↓
Authentication / Authorization
```

---

## 17.4 Security Telemetry Flow

```text
Application Activity
        ↓
Security Event
        ↓
Structured Log
        ↓
Wazuh
        ↓
Decoder
        ↓
Detection Rule
        ↓
Alert
        ↓
SOC Investigation
```

The complete data-flow model is documented in:

```text
docs/security/data-flow.md
```

---

# 18. Threat Model

The threat model uses **STRIDE** as its primary threat classification methodology.

The main categories are:

```text
Spoofing
Tampering
Repudiation
Information Disclosure
Denial of Service
Elevation of Privilege
```

Threat scenarios are evaluated according to:

```text
Risk Score = Likelihood × Impact
```

Both likelihood and impact use a scale from 1 to 5.

The threat model considers the security impact of attacks against:

- authentication,
- authorization,
- biometric recognition,
- API security,
- application integrity,
- data confidentiality,
- infrastructure availability,
- security telemetry,
- and SOC detection capabilities.

The complete threat model is maintained separately in:

```text
docs/security/threat-model.md
```

This separation allows the threat model to evolve independently from the high-level architecture while maintaining the same architectural foundation.

---

# 19. Security Assumptions

The architecture depends on a set of explicit assumptions.

The most important assumptions are:

1. The laboratory is self-hosted and controlled by the project owner.
2. Security testing is performed only against authorized laboratory assets.
3. The client is considered untrusted.
4. Authentication and authorization are enforced server-side.
5. Internal services are not intended to be directly exposed to untrusted clients.
6. Application security events should produce structured telemetry.
7. Wazuh provides the primary monitoring and detection layer.
8. Controlled attacks are used to validate both preventive and detective controls.
9. Production systems and unrelated third-party infrastructure are outside the testing scope.
10. Changes to the architecture may require the threat model and assumptions to be reviewed.

The complete assumption set is documented in:

```text
docs/security/security-assumptions.md
```

---

# 20. Network Topology

The laboratory uses a controlled virtual network.

The primary attack environment is provided by Parrot OS.

Virtual machines are managed using VirtualBox.

The network topology is intentionally kept at a simplified documentation level during this milestone.

The relevant conceptual flow is:

```text
Parrot Attacker
       ↓
Laboratory Application
       ↓
Internal Services
```

The network architecture may become more detailed during later implementation and testing milestones if additional segmentation, routing, firewalling, or service exposure becomes necessary.

For M0, the network topology is therefore treated as:

```text
Defined conceptually
+
Documented
+
Not over-engineered before implementation
```

This prevents architectural documentation from becoming more complex than the actual laboratory requirements.

---

# 21. Virtualization Environment

VirtualBox is the selected virtualization platform for the laboratory.

The environment may include:

```text
Host System
│
├── Parrot OS
│   └── Controlled Attacker
│
└── Windows Server
    └── Reusable Infrastructure / AD Laboratory
```

The existing Windows Server virtual machine is considered reusable laboratory infrastructure.

It is not required to be part of the initial biometric application architecture.

It may instead be reused in future milestones or related security scenarios where Windows infrastructure, Active Directory, identity management, or enterprise security behavior is relevant.

This allows the existing laboratory environment to evolve without unnecessarily coupling Windows Server infrastructure to the biometric application.

---

# 22. Repository Documentation Structure

The repository separates architectural documentation from security-specific documentation.

The current structure is:

```text
docs/
│
├── architecture/
│   ├── attack-surface-map.drawio.png
│   ├── data-flow-diagram.drawio.png
│   ├── docker-topology.drawio.png
│   ├── logical-architecture.drawio.png
│   ├── service-boundary-model.drawio.png
│   ├── system-architecture.drawio.png
│   ├── trust-boundaries.drawio.png
│   └── architecture-overview.md
│
└── security/
    ├── assets.md
    ├── data-flow.md
    ├── security-assumptions.md
    └── threat-model.md
```

The architecture diagrams provide visual representations.

The security documents provide detailed definitions.

This architecture overview acts as the consolidated entry point connecting both groups.

---

# 23. Architectural Decisions

The following decisions were established during M0.

## Decision 1 — Dedicated Backend

Spring Boot is used as a dedicated backend/API instead of allowing the frontend to directly control protected resources.

### Reason

The backend provides a centralized enforcement point for:

- authentication,
- authorization,
- input validation,
- API security,
- rate limiting,
- business logic,
- and telemetry.

---

## Decision 2 — Frontend Is Untrusted

Next.js is considered a presentation layer rather than a security authority.

### Reason

Frontend logic can be manipulated by an attacker.

Security decisions must therefore remain server-side.

---

## Decision 3 — Dedicated Biometric Service

CompreFace is isolated as a biometric recognition service.

### Reason

Biometric processing is security-sensitive and should remain separated from the presentation layer and authorization logic.

---

## Decision 4 — PostgreSQL Behind the Backend

PostgreSQL is accessed through Spring Boot.

### Reason

Direct frontend-to-database access would weaken the intended security boundary and complicate centralized authorization and validation.

---

## Decision 5 — Wazuh as Security Layer

Wazuh is used as the primary security monitoring and detection platform.

### Reason

The project is intended to demonstrate a complete defensive workflow rather than merely application exploitation.

---

## Decision 6 — Parrot as Controlled Attacker

Parrot OS provides the controlled offensive environment.

### Reason

The project requires reproducible adversarial testing against authorized laboratory infrastructure.

---

## Decision 7 — VirtualBox

VirtualBox is used as the virtualization platform.

### Reason

It provides a familiar and controlled environment for managing the laboratory virtual machines.

---

## Decision 8 — Documentation Before Implementation

The major architecture, assets, trust boundaries, attack surface, data flows, and threats are defined before the main implementation begins.

### Reason

Security requirements should influence the implementation rather than being added only after vulnerabilities are discovered.

---

# 24. Security Design Principles

The architecture follows several principles.

## 24.1 Least Trust

Components should not be trusted merely because they are part of the application.

The client is explicitly untrusted.

---

## 24.2 Server-Side Enforcement

Security controls that determine access must be enforced by the backend.

---

## 24.3 Separation of Responsibilities

Presentation, application logic, data storage, biometric processing, monitoring, and attack infrastructure have separate responsibilities.

---

## 24.4 Defense in Depth

Security does not depend on a single control.

The architecture combines:

```text
Authentication
+
Authorization
+
Input Validation
+
API Security
+
Rate Limiting
+
Biometric Controls
+
Logging
+
Detection
+
SOC Investigation
```

---

## 24.5 Observability

A security control that cannot be observed or validated is difficult to evaluate.

The laboratory therefore treats telemetry and detection as part of the security architecture rather than as optional operational features.

---

## 24.6 Controlled Validation

Security controls must eventually be validated through controlled attack scenarios.

The expected lifecycle is:

```text
Prevent
  ↓
Detect
  ↓
Investigate
  ↓
Respond
  ↓
Remediate
  ↓
Retest
```

---

# 25. Scope of M0

Milestone 0 establishes the architectural and security foundation.

The following areas have been defined:

```text
Repository Foundation
System Architecture
Network Topology
Docker Topology
Service Boundaries
Trust Boundaries
Attack Surface
Security Assets
Threat Model
Data Flow
Security Assumptions
```

The network topology has intentionally been kept at a simplified conceptual level.

The objective of M0 is not to implement the complete application.

The objective is to establish a sufficiently precise model so that implementation and security testing can proceed against a defined architecture.

---

# 26. M0 Completion Criteria

M0 is considered complete when the following conditions are satisfied:

- The repository structure is established.
- The major system components are identified.
- The logical architecture is documented.
- The Docker topology is defined.
- Service boundaries are documented.
- Trust boundaries are documented.
- The attack surface is identified.
- Security assets are identified.
- The threat model is defined using STRIDE.
- Data flows are documented.
- Security assumptions are explicitly stated.
- The architecture diagrams are stored in the repository.
- The architecture overview consolidates the preceding decisions.

At this point, the project has a defined architectural and security baseline.

Future milestones should therefore be evaluated against this baseline.

---

# 27. Architectural Baseline

The M0 architectural baseline can be summarized as follows:

```text
                         ┌─────────────────────┐
                         │   Parrot Attacker   │
                         │ Controlled Testing  │
                         └──────────┬──────────┘
                                    │
                                  HTTP/S
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │      Next.js        │
                         │ Presentation Layer  │
                         └──────────┬──────────┘
                                    │
                                   API
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │     Spring Boot     │
                         │                     │
                         │ Authentication      │
                         │ Authorization       │
                         │ Business Logic      │
                         │ API Security        │
                         │ Input Validation    │
                         │ Rate Limiting       │
                         │ Telemetry            │
                         └──────┬───────┬──────┘
                                │       │
                       ┌────────┘       └─────────┐
                       ▼                          ▼
              ┌────────────────┐         ┌────────────────┐
              │   PostgreSQL   │         │   CompreFace   │
              │ Application    │         │   Biometric    │
              │ Data           │         │ Recognition    │
              └────────────────┘         └────────────────┘
                                │
                                │ Security Telemetry
                                ▼
                         ┌─────────────────────┐
                         │       Wazuh         │
                         │ Collection          │
                         │ Detection           │
                         │ Alerting            │
                         └──────────┬──────────┘
                                    │
                                  Alerts
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │     SOC Analyst     │
                         │ Triage / Evidence   │
                         │ Timeline / Scope    │
                         │ Classification      │
                         │ Response / Retest   │
                         └─────────────────────┘
```

This diagram represents the conceptual security architecture established during M0.

The exact implementation details may evolve during subsequent milestones, but any architectural change that affects security boundaries, assets, trust relationships, data flows, or threat assumptions should trigger a review of the relevant M0 documentation.

---

# 28. Final M0 Statement

Milestone 0 establishes the Biometric Security Lab as a controlled environment in which application security and defensive security are developed together.

The architecture intentionally separates:

```text
Presentation
Application Security
Data Storage
Biometric Processing
Security Monitoring
Attack Infrastructure
SOC Investigation
```

The most important architectural principle established during this milestone is that the laboratory is not simply an application with security features.

It is a security validation environment.

Its purpose is to allow the following relationship to be demonstrated:

```text
Application
     ↓
Security Control
     ↓
Attack
     ↓
Observable Behavior
     ↓
Telemetry
     ↓
Detection
     ↓
Investigation
     ↓
Remediation
     ↓
Retest
```

This baseline will serve as the architectural reference for the subsequent milestones of the project.