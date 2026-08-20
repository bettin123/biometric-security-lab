# Threat Model

## 1. Purpose

This document defines the initial threat model for the Biometric Security Lab.

The objective is to identify the primary security threats affecting the application's architecture, infrastructure, authentication and biometric flows, authorization model, APIs, data storage, and security monitoring capabilities.

The threat model establishes the security scenarios that will later be validated through controlled attack simulations and SOC investigations.

This document represents the initial design-stage threat model. It should be reviewed and updated when significant architectural or implementation changes are introduced.

---

# 2. Scope

The threat model covers the following components:

- Next.js web application.
- Spring Boot backend/API.
- PostgreSQL database.
- CompreFace biometric recognition service.
- Wazuh monitoring and detection infrastructure.
- Docker-based application services.
- VirtualBox laboratory environment.
- Kali/Parrot attack environment.
- Application authentication and authorization mechanisms.
- Biometric recognition flows.
- Application API endpoints.
- Security telemetry and logging pipeline.

The threat model does not cover unrelated external systems or production infrastructure.

---

# 3. Security Objectives

The primary security objectives are:

### Confidentiality

Protect:

- User information.
- Authentication credentials and password hashes.
- Authorization information.
- Biometric-related information.
- Internal service information.
- Security telemetry containing sensitive context.

### Integrity

Protect against:

- Unauthorized modification of user data.
- Unauthorized modification of authorization data.
- Manipulation of application state.
- Unauthorized modification of security telemetry.
- Manipulation of biometric-related configuration or data.

### Availability

Protect:

- Application availability.
- Authentication availability.
- Biometric recognition availability.
- API availability.
- Monitoring and detection capabilities.

### Accountability

Maintain sufficient telemetry to determine:

- Who performed an action.
- What action occurred.
- When it occurred.
- Which resource was involved.
- Where the activity originated.
- Whether the action was allowed or denied.

---

# 4. Trust Model

The system is divided into multiple trust zones.

## 4.1 Client Zone

The client is considered untrusted.

Requests originating from the client may contain:

- Malicious input.
- Manipulated parameters.
- Forged identifiers.
- Unauthorized requests.
- Automated requests.
- Attempted authentication abuse.

Client-side validation is therefore not considered a sufficient security control.

---

## 4.2 Application Zone

The application zone contains:

- Next.js.
- Spring Boot.

Spring Boot represents the authoritative backend security boundary.

Security-critical decisions must be enforced server-side.

---

## 4.3 Internal Services Zone

This zone contains:

- PostgreSQL.
- CompreFace.

These services should not be directly accessible from untrusted clients.

Access should occur through controlled application flows.

---

## 4.4 Security Monitoring Zone

Wazuh represents the monitoring and detection layer.

Its responsibilities include:

- Collecting security telemetry.
- Parsing events.
- Applying detection rules.
- Correlating relevant activity.
- Generating alerts.

---

## 4.5 Attack Zone

Kali/Parrot represents the controlled adversarial environment.

Its purpose is to generate authorized attack traffic against laboratory assets.

---

# 5. Threat Actors

## 5.1 Unauthenticated External Attacker

An attacker without valid credentials attempting to:

- Discover application behavior.
- Enumerate users or resources.
- Abuse authentication endpoints.
- Exploit application vulnerabilities.
- Abuse APIs.

---

## 5.2 Authenticated Malicious User

A user possessing valid credentials but attempting to:

- Access unauthorized resources.
- Escalate privileges.
- Abuse APIs.
- Access another user's objects.
- Abuse biometric functionality.

---

## 5.3 Compromised Identity

A legitimate account whose credentials or authentication context have been compromised.

The attacker may attempt to:

- Access protected resources.
- Perform privileged actions.
- Abuse legitimate application functionality.
- Evade detection by operating through an apparently valid identity.

---

## 5.4 Malicious or Manipulated Client

A client capable of modifying requests independently of the intended frontend behavior.

This threat actor is particularly relevant because the backend must not assume that requests originated from the legitimate Next.js interface.

---

# 6. Threat Identification

The initial threat model defines the following primary threats.

---

## TH-01 — Authentication Abuse

### Description

An attacker repeatedly attempts to authenticate using invalid credentials or manipulated authentication requests.

### Examples

- Credential brute force.
- Password guessing.
- Repeated login attempts.
- Automated authentication requests.

### Potential Impact

- Account compromise.
- Account lockout.
- Authentication service degradation.
- Increased attack surface.

### Relevant Components

- Next.js.
- Spring Boot authentication endpoints.
- PostgreSQL user records.
- Wazuh telemetry.

### Expected Controls

- Authentication controls.
- Rate limiting.
- Secure credential handling.
- Authentication event logging.
- Wazuh detection rules.

---

## TH-02 — Account / Resource Enumeration

### Description

An attacker attempts to determine whether users, accounts, resources, or endpoints exist.

### Examples

- Username enumeration.
- Resource ID enumeration.
- Distinguishing valid and invalid API responses.
- Identifying administrative endpoints.

### Potential Impact

- Information disclosure.
- Improved attacker reconnaissance.
- Facilitation of subsequent attacks.

### Relevant Components

- Next.js.
- Spring Boot API.
- PostgreSQL.
- API responses.

### Expected Controls

- Consistent error handling.
- Authorization controls.
- Response minimization.
- Input validation.
- Security telemetry.

---

## TH-03 — Broken Authorization

### Description

An authenticated user attempts to perform an action outside the permissions associated with their identity or role.

### Examples

- Regular user accessing administrative functionality.
- Unauthorized modification of protected resources.
- Privilege escalation through manipulated requests.

### Potential Impact

- Unauthorized access.
- Privilege escalation.
- Data modification.
- Administrative compromise.

### Relevant Components

- Spring Boot.
- Authorization layer.
- PostgreSQL.
- Protected API endpoints.

### Expected Controls

- Server-side authorization.
- Role/permission enforcement.
- Deny-by-default behavior.
- Authorization event logging.

---

## TH-04 — Broken Object-Level Authorization / IDOR

### Description

An authenticated user attempts to access or modify an object belonging to another user by manipulating an object identifier.

### Example

```text
User A

GET /api/resources/101