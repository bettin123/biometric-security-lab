# M0.8 — Security Assets

## Purpose

This document identifies and classifies the assets that are relevant to the security of the biometric access-control laboratory.

The objective is to determine which resources require protection and to evaluate the potential impact of their compromise across the three fundamental security properties:

- Confidentiality (C)
- Integrity (I)
- Availability (A)

The asset inventory will serve as an input for the M0.9 Threat Model and for subsequent attack scenarios, detection engineering activities, and SOC investigations.

---

## Classification

| Rating | Meaning |
|---|---|
| Low | Limited security impact |
| Medium | Noticeable impact, but limited scope |
| High | Significant impact on the system or security posture |
| Critical | Severe impact that may compromise a core security objective or the overall environment |

---

## Asset Inventory

| ID | Asset | Category | Confidentiality | Integrity | Availability |
|---|---|---|---|---|---|
| A-01 | User Accounts | Identity | High | High | Medium |
| A-02 | Authentication Credentials | Identity | Critical | Critical | High |
| A-03 | Roles & Permissions | Authorization | High | Critical | High |
| A-04 | Biometric Data | Sensitive Data | Critical | Critical | High |
| A-05 | Next.js Frontend | Application | Medium | High | Medium |
| A-06 | Spring Boot API | Application | High | Critical | Critical |
| A-07 | PostgreSQL Database | Data | Critical | Critical | Critical |
| A-08 | CompreFace | Biometric Service | Critical | Critical | High |
| A-09 | Security Telemetry | Security | High | Critical | Critical |
| A-10 | Wazuh Security Infrastructure | Detection | High | Critical | Critical |
| A-11 | Docker Infrastructure | Infrastructure | High | Critical | High |
| A-12 | Virtual Machines / Hosts | Infrastructure | High | Critical | Critical |

---

# Asset Descriptions

## A-01 — User Accounts

User accounts represent the identities registered in the biometric access-control application.

They include information required to associate a user with their account state and assigned permissions.

### Security relevance

Unauthorized access to an account may allow an attacker to impersonate a legitimate user or use the account as a starting point for further attacks.

---

## A-02 — Authentication Credentials

Authentication credentials include the mechanisms used to authenticate users and maintain authenticated sessions.

Examples include:

- Passwords
- Authentication tokens
- Session information
- Administrative credentials

### Security relevance

Credential compromise may allow unauthorized authentication and impersonation.

Credential integrity is also critical because manipulated authentication material may result in unauthorized access.

---

## A-03 — Roles & Permissions

Roles and permissions determine which operations each user is authorized to perform.

Examples include administrative and standard user privileges.

### Security relevance

Unauthorized modification of permissions could allow privilege escalation or authorization bypass.

This asset is therefore particularly relevant to scenarios involving:

- Broken access control
- Privilege escalation
- Authorization bypass
- IDOR / BOLA

---

## A-04 — Biometric Data

Biometric data represents the information processed by the facial-recognition component to associate a biometric identity with an application user.

### Security relevance

Biometric information is treated as a critical confidentiality and integrity asset.

Unauthorized disclosure may expose sensitive biometric information, while manipulation may result in incorrect identity recognition and potentially unauthorized access.

---

## A-05 — Next.js Frontend

The Next.js frontend provides the user-facing interface of the application.

### Security relevance

The frontend represents an exposed application component and may be targeted through:

- Manipulation of client requests
- Cross-site scripting
- Unauthorized functionality access
- Malicious input
- API abuse originating from the client layer

---

## A-06 — Spring Boot API

The Spring Boot backend provides the primary application and security logic.

Its responsibilities include:

- Authentication
- Authorization
- Business logic
- API security
- Input validation
- Rate limiting
- Security telemetry generation
- Communication with backend services

### Security relevance

The API represents one of the most critical application assets because compromise of its integrity may affect multiple security controls simultaneously.

---

## A-07 — PostgreSQL Database

PostgreSQL stores application information required for the operation of the system.

This may include:

- Users
- Roles
- Permissions
- Authentication-related information
- Application state
- Security-related metadata

### Security relevance

Database compromise may affect confidentiality, integrity, and availability simultaneously.

---

## A-08 — CompreFace

CompreFace provides the facial-recognition capabilities used by the application.

### Security relevance

The service processes biometric information and represents a critical internal service.

Security concerns include:

- Unauthorized API access
- Service manipulation
- Configuration compromise
- Biometric processing abuse
- Availability disruption
- Compromise of service credentials

---

## A-09 — Security Telemetry

Security telemetry consists of structured security events generated by the application during normal operation and during controlled security testing.

Examples include:

- Successful authentication
- Authentication failures
- Unknown face recognition
- Administrative activity
- Suspicious API requests
- Rate-limit violations
- Security-relevant application events

### Security relevance

Telemetry is a critical defensive asset because it provides the evidence required for detection and investigation.

Its integrity must be preserved because manipulation of fields such as timestamps, source IP addresses, identities, event types, or results could affect investigations.

Its availability is also critical because loss of telemetry may result in attacks becoming invisible to the detection layer.

---

## A-10 — Wazuh Security Infrastructure

Wazuh provides the security monitoring and detection layer of the laboratory.

It processes security telemetry through components such as:

- Log collection
- Decoders
- Detection rules
- Alerts

### Security relevance

Compromise or disruption of Wazuh may reduce or eliminate the laboratory's ability to detect and investigate malicious activity.

Detection logic, decoders, and rules are considered components of this asset rather than independent assets.

---

## A-11 — Docker Infrastructure

Docker provides the containerized execution environment for the application services.

This includes:

- Containers
- Service configuration
- Internal networking
- Volumes
- Secrets
- Container privileges

### Security relevance

Compromise of Docker configuration or container isolation may allow an attacker to affect multiple application services.

---

## A-12 — Virtual Machines / Hosts

Virtual machines and their supporting hosts provide the infrastructure on which the laboratory components execute.

These systems include the application, defensive infrastructure, and controlled attack environment.

### Security relevance

Compromise of a host or virtual machine may provide an attacker with access to multiple layers of the laboratory and potentially bypass application-level security controls.

---

# Asset Relationships

The assets are not isolated. Several security dependencies exist between them.

```text
User Accounts
      │
      ▼
Authentication Credentials
      │
      ▼
Roles & Permissions
      │
      ▼
Spring Boot API
      │
      ├──────────► PostgreSQL
      │
      └──────────► CompreFace
                         │
                         ▼
                  Biometric Data

Application Activity
      │
      ▼
Security Telemetry
      │
      ▼
Wazuh
      │
      ▼
Detection / Alerting
      │
      ▼
SOC Investigation



# M0.8 — Security Assets

## 1. Purpose

[Qué estamos haciendo en este documento]

## 2. Classification Methodology

[Qué significa C / I / A y qué significa
Low / Medium / High / Critical]

## 3. Asset Inventory

[LA TABLA DE LOS 12 ASSETS]

## 4. Asset Descriptions

[Las descripciones A-01 → A-12]

## 5. Asset Relationships

[Los diagramas ASCII de relaciones]

## 6. Scope Notes

[Las decisiones sobre secrets, admin functions,
Wazuh rules, etc.]


Impact levels:

- Low
- Medium
- High
- Critical

## Asset Inventory

| ID | Asset | Category | Confidentiality | Integrity | Availability |
|---|---|---|---|---|---|
| A-01 | User Accounts | Identity | High | High | Medium |
| A-02 | Authentication Credentials | Identity | Critical | Critical | High |
| A-03 | Roles & Permissions | Authorization | High | Critical | High |
| A-04 | Biometric Data | Sensitive Data | Critical | Critical | High |
| A-05 | Next.js Frontend | Application | Medium | High | Medium |
| A-06 | Spring Boot API | Application | High | Critical | Critical |
| A-07 | PostgreSQL Database | Data | Critical | Critical | Critical |
| A-08 | CompreFace | Biometric Service | Critical | Critical | High |
| A-09 | Security Telemetry | Security | High | Critical | Critical |
| A-10 | Wazuh Security Infrastructure | Detection | High | Critical | Critical |
| A-11 | Docker Infrastructure | Infrastructure | High | Critical | High |
| A-12 | Virtual Machines / Hosts | Infrastructure | High | Critical | Critical |

## Asset Descriptions

### A-01 — User Accounts

User identities and account state used by the application.

This includes the identity associated with application users and their relationship with authentication and authorization mechanisms.

### A-02 — Authentication Credentials

Credentials and authentication artifacts used to establish user identity.

This may include passwords, authentication tokens, sessions, and service credentials where applicable.

### A-03 — Roles & Permissions

Authorization information that determines which operations a user is allowed to perform.

Compromise of this asset may result in unauthorized privilege escalation or access to restricted functionality.

### A-04 — Biometric Data

Biometric information processed by the facial recognition subsystem.

Unauthorized disclosure or manipulation of biometric information may affect both user privacy and the correctness of authentication or access-control decisions.

### A-05 — Next.js Frontend

The web application interface through which users interact with the system.

The frontend represents an externally exposed application component and an entry point for user and attacker-controlled requests.

### A-06 — Spring Boot API

The primary application backend responsible for authentication, authorization, business logic, API security, input validation, rate limiting, and security telemetry.

Compromise of the API could affect multiple security controls simultaneously.

### A-07 — PostgreSQL Database

The database containing application information required for system operation, including users, roles, permissions, authentication-related information, and application state.

### A-08 — CompreFace

The biometric recognition service used by the application to perform facial recognition operations.

Its availability, configuration, access controls, and biometric processing capabilities are security-relevant.

### A-09 — Security Telemetry

Structured security events generated by the application and supporting services.

Telemetry provides evidence for detection and investigation and therefore requires protection against unauthorized modification, deletion, or disruption.

### A-10 — Wazuh Security Infrastructure

The defensive infrastructure responsible for collecting, processing, detecting, and alerting on security telemetry.

This includes the mechanisms used for log processing, decoders, detection rules, and alert generation.

### A-11 — Docker Infrastructure

The containerized execution environment and configuration used to isolate and connect application services.

This includes containers, service configuration, internal networking, volumes, and relevant secrets.

### A-12 — Virtual Machines / Hosts

The virtual and physical infrastructure supporting the laboratory environment.

Compromise of the underlying hosts or virtual machines could provide an attacker with access to multiple application or security components.

## Asset Relationships

The assets are not independent. Compromise of one asset may affect others.

Examples:

```text
Authentication Credentials
        ↓
User Account
        ↓
Roles & Permissions
        ↓
Authorized Application Access