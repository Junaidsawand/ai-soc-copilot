# Production Deployment Guide: AI SOC Copilot

## 1. Executive Summary
This document specifies the deployment architecture and operational runbooks for the AI SOC Copilot MVP. The deployment strategy favors a **Containerized Modular Monolith**, optimized for low operational overhead via modern Platform-as-a-Service (PaaS) providers, while maintaining stringent enterprise security postures.

---

## 2. Infrastructure & Hosting

### 2.1 Target Hosting Environment
The MVP explicitly targets managed PaaS providers (e.g., Render, Railway, or AWS App Runner) to minimize DevOps overhead for the solo-founder/lean team constraint.

- **Frontend:** Flutter Web compiled to static assets (`HTML/JS/CSS`), distributed via a global CDN.
- **Backend:** FastAPI Python application running via ASGI server (`Uvicorn` with `Gunicorn` workers).
- **Database:** Managed PostgreSQL (16+) instance.

### 2.2 Containerization (Docker)
The backend is packaged as an OCI-compliant Docker container, ensuring absolute environmental parity between Development, Staging, and Production.

- **Base Image:** `python:3.11-slim` (Minimizes attack surface).
- **Non-Root Execution:** The Dockerfile explicitly creates a non-root `appuser` to run the FastAPI process, adhering to the principle of least privilege.
- **Immutability:** Docker images are tagged with the specific Git commit SHA. `latest` tags are avoided in production deployment manifests.

---

## 3. Environment & Secrets Management

Security is paramount. No secrets are ever committed to version control.

### 3.1 Environment Variables
Application configuration is injected dynamically at runtime via OS-level environment variables (Twelve-Factor App methodology). 
- `DATABASE_URL`: Standard PostgreSQL connection string.
- `OPENAI_API_KEY`: API token for LLM inference.
- `ENVIRONMENT`: Defines the runtime context (`development`, `staging`, `production`).
- `LOG_LEVEL`: Configures logging verbosity (`INFO` or `DEBUG`).

### 3.2 Secrets Management
In production, environment variables are managed by the PaaS provider's integrated Secrets Manager (e.g., AWS Parameter Store, Render Secret Files). These vaults encrypt variables at rest and only decrypt them into the container's ephemeral memory upon startup.

---

## 4. Network Security & HTTPS

### 4.1 Transport Layer Security (TLS/SSL)
- **HTTPS Enforcement:** All HTTP traffic is forcefully redirected to HTTPS (Port 443).
- **Termination:** TLS termination occurs at the PaaS Load Balancer/CDN layer. Internal container traffic operates over private virtual networks.

### 4.2 Cross-Origin Resource Sharing (CORS)
To prevent Cross-Site Request Forgery (CSRF) and unauthorized API usage, the backend CORS middleware is strictly whitelisted to the production frontend domain (e.g., `https://aisoccopilot.com`). Wildcards (`*`) are explicitly prohibited in the production environment.

---

## 5. CI/CD Pipeline

Continuous Integration and Continuous Deployment (CI/CD) is fully automated using GitHub Actions.

### 5.1 CI Pipeline (On Pull Request)
1. Code formatting and linting verification (`Black`, `Ruff`).
2. SAST Scanning (`Bandit`) to detect hardcoded secrets.
3. Automated unit and integration test execution.
4. Ephemeral PostgreSQL database spun up to verify Alembic migrations apply cleanly.

### 5.2 CD Pipeline (On Merge to `main`)
1. Executes final regression test suite.
2. Builds the Docker Image and tags it with the Git Commit SHA.
3. Pushes the image to a private Container Registry.
4. Triggers an automated rolling deployment on the Staging environment.
5. Awaits manual approval (via GitHub Environments) to promote the image to Production.

---

## 6. Monitoring & Logging

### 6.1 Observability
- **Health Probes:** The PaaS load balancer continuously polls the `GET /api/v1/health` endpoint. If the backend fails to respond 3 consecutive times, the container is automatically restarted.
- **Application Logging:** Standardized JSON formatting is utilized for standard output (`stdout`). Logs include a unique `trace_id` to track investigations across the asynchronous LLM workflow.
- **Error Tracking:** (Future) Integration with Sentry for real-time alerting on backend exceptions and UI crashes.

---

## 7. Disaster Recovery & Backups

### 7.1 Backup Strategy
- **Automated Snapshots:** The managed PostgreSQL database performs daily automated point-in-time snapshots.
- **Retention:** Snapshots are retained for a minimum of 7 days (configurable based on compliance needs).
- **Immutable Storage:** (Future) Archiving of `uploaded_alerts` to WORM-compliant AWS S3 buckets for forensic retention.

### 7.2 Disaster Recovery (RTO/RPO)
Because the application state is entirely contained within the managed database, and infrastructure is defined as code (or managed by PaaS):
- **Recovery Time Objective (RTO):** < 2 Hours (Time required to deploy containers to a new region and restore the DB snapshot).
- **Recovery Point Objective (RPO):** < 24 Hours (Based on the daily snapshot frequency).

---

## 8. Rollback Procedures

If a newly deployed version introduces critical failures (e.g., 500 errors spike, database migrations fail), the following runbook is executed:

1. **Halt Deployment:** Freeze the CI/CD pipeline to prevent further changes.
2. **Revert Image:** In the PaaS dashboard (or via CLI), instruct the orchestrator to deploy the previous, known-good Docker image tag.
3. **Database Considerations:** If the deployment included a breaking schema migration, the DBA must manually run `alembic downgrade -1` before the old image is spun up. *(Note: Migrations should ideally be written as backward-compatible to avoid this scenario).*
4. **Post-Mortem:** Once stability is restored, perform root cause analysis before unfreezing the pipeline.
