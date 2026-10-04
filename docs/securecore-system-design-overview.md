# SecureCore System Design

## 1. System Purpose
SecureCore is a reusable security framework for Java enterprise applications, with a primary focus on banking and financial systems. It centralizes common security concerns so business-specific applications can focus on their domain logic.

## 2. Design Principles
- Security by default
- Least privilege
- Separation of business logic and security logic
- Stateless authentication with JWT
- Centralized policy management
- Multi-instance compatibility
- Auditability and traceability

## 3. Core Architecture
### 3.1 Layers
```text
Client
  │
  ▼
API Gateway / Load Balancer
  │
  ▼
Applications (App-01, App-02, App-03)
  │
  ▼
SecureCore Framework
  ├── Authentication
  ├── Authorization
  ├── MFA
  ├── Token Management
  ├── Session & Device Security
  ├── Cryptography
  ├── Rate Limiting
  ├── Audit & Security Events
  └── Transaction Security
  │
  ▼
MySQL 8.x + Redis
```

## 4. Major Modules
### Authentication Module
Handles login, logout, credential validation, account status, brute-force tracking, and session creation.

### Authorization Module
Handles RBAC and permission evaluation using roles and resource permissions.

### MFA Module
Supports TOTP, OTP, and recovery codes.

### Token Module
Handles access token generation, refresh token rotation, revocation, and expiry.

### Session & Device Management
Tracks user sessions, device trust, revocation, and session termination.

### Cryptography Module
Provides hashing, encryption, signing, and secret management abstraction.

### Rate Limiting Module
Protects APIs from abuse through IP, user, device, and API-level limits.

### Audit & Security Event Module
Captures security-sensitive actions with traceable events and correlation IDs.

### Transaction Security Module
Manages idempotency, risk checks, signing, transaction limits, and financial authorization logic.

## 5. Data Model Overview
```text
users
 ├── roles (via user_roles)
 ├── permissions (via user_roles + role_permissions)
 ├── sessions
 ├── devices
 ├── refresh_tokens
 ├── mfa_configurations
 ├── login_attempts
 ├── account_locks
 ├── security_events
 ├── audit_logs
 └── transaction_security
```

## 6. Technical Standards
- Java 21
- Spring Boot 3.x
- Spring Security 6.x
- MySQL 8.x
- Redis
- Flyway
- JPA/Hibernate
- Maven
- JUnit 5 + Testcontainers

## 7. Security Design Details
### Password Protection
- Use Argon2id
- Unique salt per user/password
- Never store plaintext values
- Enforce policy rules

### JWT Design
- Short-lived access tokens
- Long-lived refresh tokens
- Refresh token rotation
- Device association
- Revoke tokens on security incidents

### MFA Design
- Primary factor: password
- Secondary factor: TOTP
- Recovery flow for locked users
- Secret encryption at rest

### Audit Design
- Generate events independently from app logs
- Associate each event with session/user/correlation ID
- Log the result and severity of the security action
- Redact secrets before writing logs

## 8. API Design Principles
- Base path: `/api/v1`
- Bearer token auth
- Standard success/error payload format
- Correlation ID required on every request
- Secure headers enabled as needed
- Idempotency key required for financial mutations

## 9. Application Boundaries
SecureCore should not own the complete financial ledger. It supplies:
- Authorization checks
- Idempotency enforcement
- Risk management
- Limits and signing checks
- Audit trail

The consuming bank application owns the actual financial transaction records and ledger operations.

## 10. Example Workflow
### Authentication workflow
1. Client sends username + password
2. System validates status and password
3. System checks failed attempts and lockout policy
4. MFA verification if required
5. Session created and tokens issued
6. Security event recorded

### Transaction workflow
1. Client submits request with Idempotency-Key
2. Authentication and authorization validation
3. Risk and limit checks
4. Canonicalization and signature check
5. Domain business service executes ledger operation
6. Security metadata and audit log stored

## 11. Deployment Model
- Multi-instance Spring Boot services behind a load balancer
- Redis for distributed rate limits and short-lived state
- MySQL for persistent identity, session, and audit data
- Environment-managed secrets via KMS/HSM/secrets manager

## 12. Outcome
This design ensures SecureCore can be reused across multiple Java enterprise systems while preserving strong security, clear separation of concerns, and maintainability.
