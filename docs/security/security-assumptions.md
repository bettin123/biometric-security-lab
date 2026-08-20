# Security Assumptions

## 1. Purpose

This document defines the assumptions under which the architecture and threat model of the Biometric Security Lab are considered valid.

These assumptions establish the expected characteristics of the laboratory environment, application architecture, identity model, infrastructure, monitoring capabilities, and security testing process.

If any of these assumptions change, the corresponding architectural or threat-model decisions should be reviewed.

---

## 2. Environment Assumptions

- The project is implemented as a self-hosted security laboratory.
- Virtual machines are managed using VirtualBox.
- The laboratory environment is controlled by the project owner.
- Security testing is performed only against systems belonging to the laboratory.
- Production systems, real credentials, and unrelated third-party infrastructure are outside the scope of testing.
- The laboratory is designed to support controlled adversarial simulations.

---

## 3. Identity and Authentication Assumptions

- Users are expected to authenticate before accessing protected resources.
- Authentication establishes the identity used by subsequent authorization decisions.
- Client-provided identity information is not considered inherently trustworthy.
- Authentication failures are considered security-relevant events.
- Biometric recognition may contribute to establishing user identity within the authentication flow.
- Biometric recognition does not independently determine authorization privileges.
- Credentials and authentication secrets must not be stored or exposed in plaintext.

The conceptual identity flow is:

Client
→ Authentication
→ Identity
→ Authorization

For biometric authentication:

Biometric Recognition
→ Identity
→ Authentication
→ Authorization

---

## 4. Network Assumptions

- The laboratory operates within a controlled virtual network.
- Attack traffic generated during testing originates from designated attack systems such as Kali or Parrot.
- The attack infrastructure is intended to interact with the laboratory services only.
- Internal services such as PostgreSQL and CompreFace are not intended to be directly exposed to untrusted clients.
- Network segmentation and service exposure are controlled as part of the laboratory architecture.

The intended security boundary is:

Internet / External Environment
→ Laboratory Boundary
→ Application Services
→ Internal Services

---

## 5. Application Assumptions

- Next.js provides the web application/frontend layer.
- Spring Boot provides the dedicated backend/API layer.
- The backend is considered the authoritative security boundary for application requests.
- Client-side controls are not considered sufficient security controls.
- Authentication and authorization decisions are enforced server-side.
- Input received from clients is considered untrusted.
- PostgreSQL is accessed through controlled backend operations.
- Direct database access from the frontend is outside the intended architecture.
- Security-relevant application events should generate structured telemetry.

The intended request flow is:

Client
→ Next.js
→ Spring Boot
→ Security Controls
→ Internal Services

---

## 6. Infrastructure Assumptions

The laboratory infrastructure is expected to contain:

- Application containers.
- PostgreSQL as the application database.
- CompreFace as the biometric recognition service.
- Wazuh as the security monitoring and detection platform.
- Virtual machines providing the laboratory and attack environments.

Infrastructure components are considered part of the security boundary of the laboratory.

---

## 7. Monitoring and Logging Assumptions

- Security-relevant application activity should generate structured telemetry.
- Application events should contain sufficient contextual information to support investigation.
- Logs should avoid unnecessary exposure of credentials, tokens, raw biometric information, or other sensitive data.
- Wazuh is responsible for collecting and processing relevant security telemetry.
- Detection logic is primarily implemented within the monitoring/detection layer rather than embedded entirely within the application.
- A security event does not automatically constitute a security incident.
- Detection results require analysis and investigation.

The intended monitoring pipeline is:

Application Activity
→ Security Event
→ Structured Log
→ Wazuh
→ Decoder
→ Detection Rule
→ Alert
→ Investigation

---

## 8. Security Testing Assumptions

- Security testing is performed exclusively against authorized laboratory assets.
- Attack scenarios are defined before execution.
- Attacks are performed in a controlled and reproducible manner.
- Testing is intended to generate observable application behavior and security telemetry.
- Attack scenarios should be connected to previously identified threats.
- Successful testing includes both offensive execution and defensive observation.

The intended security validation cycle is:

Defined Threat
→ Attack Scenario
→ Controlled Execution
→ Application Activity
→ Telemetry
→ Detection
→ Investigation
→ Remediation
→ Retest

---

## 9. Assumption Review

These assumptions are part of the initial threat model and architecture definition.

If the implementation introduces changes to:

- network topology,
- authentication mechanisms,
- biometric processing,
- service exposure,
- data storage,
- logging architecture,
- monitoring infrastructure, or
- testing methodology,

the affected assumptions and threat-model decisions should be reviewed.

---

## 10. Security Principle

The laboratory follows the principle that security controls must be enforced at the appropriate trust boundary and validated through observable behavior.

The project therefore treats:

- the client as untrusted,
- backend authorization as authoritative,
- internal services as protected resources,
- security telemetry as a security-critical capability, and
- controlled attacks as validation mechanisms for both preventive and detective controls.