# Security Architecture: AI SOC Copilot

## 1. Executive Summary
This document outlines the security architecture and defensive posture for the AI SOC Copilot platform. As a cybersecurity product, the system is designed with a "Security by Design" ethos, enforcing strict boundaries around data ingestion, LLM orchestration, and infrastructure deployment to mitigate risk and ensure operational integrity.

---

## 2. Threat Model & Attack Surface

### 2.1 Attack Surface
The MVP attack surface is intentionally minimized:
1. **Public API (FastAPI):** Exposes `POST /investigations` for JSON uploads.
2. **Web Frontend (Flutter):** Delivered as static assets to client browsers.
3. **LLM Egress (OpenAI):** Outbound API requests to the external AI provider.

### 2.2 Threat Modeling (STRIDE)
| Threat | Description | Mitigation |
|---|---|---|
| **Spoofing** | Unauthorized API access. | MVP limits exposure via strict CORS. Future: JWT Bearer Tokens. |
| **Tampering** | Modifying ingested alert data. | Raw alerts are stored immutably as `JSONB`. Database updates only modify specific state columns. |
| **Repudiation** | Denying an action occurred. | Mandatory, immutable logging to `audit_logs` for all state changes. |
| **Information Disclosure**| Data leakage via API errors. | "Zero Stack Trace" policy. Broad 500 errors returned to client; detailed traces logged securely internally. |
| **Denial of Service** | Volumetric JSON bombs. | Strict 10MB file size limit enforced at the ingress layer. |
| **Elevation of Privilege**| Container breakout. | Backend executes as a non-root `appuser` within Docker. |

---

## 3. Identity & Access Management (IAM)

### 3.1 Authentication
- **MVP Phase:** To eliminate friction during early testing, active user authentication is deferred. The application assumes a single-tenant deployment model.
- **Future State (V1.5):** The API is architected to support OAuth 2.0 and OpenID Connect (OIDC). Authentication will rely on JWT (JSON Web Tokens) passed via `Authorization: Bearer <token>` headers.

### 3.2 Authorization
- **Future RBAC:** The database schema inherently includes `owner_id` fields on all core tables to facilitate future Row-Level Security (RLS) policies and Role-Based Access Control (Admin vs. Analyst).

---

## 4. API & Application Security

### 4.1 Input Validation
The most critical defense mechanism is strict schema validation.
- **Pydantic Validation:** All incoming data is cast and validated against strict Python type hints. Extraneous JSON keys are dropped.
- **File Constraints:** The `/investigations` endpoint strictly enforces `Content-Type: multipart/form-data` and `.json` file extensions.

### 4.2 Network Defenses
- **CORS:** Cross-Origin Resource Sharing is tightly locked to the exact domain of the deployed frontend. Wildcards (`*`) are explicitly forbidden in production.
- **Rate Limiting:** (Future/PaaS dependent) Protection against API abuse via 429 Too Many Requests responses.

### 4.3 LLM Prompt Injection Defense
- **Strict Grounding:** The prompt templates rigidly instruct the LLM to process the injected telemetry as *evidence only*, preventing the LLM from treating user-controlled JSON values as executable instructions.
- **Schema Enforcement:** By forcing the LLM to output rigid JSON, the model's ability to execute complex injection payloads is severely neutered.

---

## 5. Data Security & Encryption

### 5.1 Data in Transit
- **TLS 1.2+:** All external and internal HTTP traffic is encrypted. The PaaS load balancer terminates TLS, ensuring no plaintext data transverses the public internet.

### 5.2 Data at Rest
- **Database Encryption:** Leverages the native at-rest encryption provided by the managed PostgreSQL hosting provider (e.g., AWS KMS-backed RDS storage).
- **Ephemeral Processing:** Reports (PDF/Markdown) are generated dynamically in memory and streamed to the user; they are never persisted to disk.

---

## 6. Secrets Management

- **Twelve-Factor Compliance:** Credentials (DB passwords, OpenAI API Keys) are strictly forbidden in the Git repository.
- **Injection:** Secrets are securely injected as environment variables at runtime by the PaaS Secrets Manager.
- **SAST Scanning:** GitHub Actions runs `Bandit` to proactively scan for accidentally committed secrets during CI/CD.

---

## 7. Secure Coding & OWASP Alignment

The development lifecycle integrates the OWASP Top 10 recommendations:

| OWASP Category | AI SOC Copilot Mitigation |
|---|---|
| **A01: Broken Access Control** | CORS enforcement; Future JWT integration. |
| **A02: Cryptographic Failures** | Mandated HTTPS/TLS; Provider-managed DB encryption. |
| **A03: Injection** | SQLAlchemy ORM prevents SQLi. Prompt templating prevents LLMi. |
| **A04: Insecure Design** | Threat modeling completed prior to development (this document). |
| **A05: Security Misconfiguration** | Dockerized infrastructure ensures deterministic environments. |
| **A06: Vulnerable Components** | CI/CD pipeline scans `requirements.txt` via Dependabot. |
| **A09: Security Logging** | Immutable `audit_logs` table tracking all lifecycle events. |

---

## 8. Logging & Compliance

- **Sanitized Logging:** Application `stdout` logs omit Personally Identifiable Information (PII) and raw authentication tokens.
- **Audit Ledger:** The `audit_logs` table serves as a permanent, append-only ledger for compliance tracking, satisfying basic forensic requirements for incident investigations.
