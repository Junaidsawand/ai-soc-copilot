# Production Deployment Guide: AI SOC Copilot

## 1. Executive Summary
This document specifies the deployment architecture, configuration parameters, and operational runbooks for the AI SOC Copilot platform. As a cybersecurity product, the deployment strategy prioritizes an immutable, containerized infrastructure (Docker) optimized for rapid scaling and low operational overhead via modern Platform-as-a-Service (PaaS) providers.

---

## 2. Infrastructure Architecture

The system utilizes a **Modular Monolith** pattern delivered via OCI-compliant containers, ensuring absolute environmental parity between Development, Staging, and Production.

### 2.1 Deployment Topology (Production)

```mermaid
graph TD
    Client[End User/Browser] --> |HTTPS / TLS 1.3| CDN[Global CDN / PaaS Load Balancer]
    
    subgraph "Managed PaaS Environment (VPC)"
        CDN --> |Port 80| Frontend[Flutter Web Container\nStatic Assets]
        CDN --> |Port 443 -> 8000| Backend[FastAPI Backend Container\nUvicorn + Gunicorn Workers]
        
        Backend --> |Connection Pool| DB[(Managed PostgreSQL 16+)]
        Backend -.-> |Secret Injection| Vault[PaaS Secrets Manager]
    end
    
    subgraph "External Integrations"
        Backend --> |REST API / HTTPS| OpenAI[OpenAI API]
    end
    
    classDef container fill:#e1f5fe,stroke:#0277bd,stroke-width:2px;
    classDef database fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef external fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    
    class Frontend,Backend container;
    class DB database;
    class OpenAI external;
```

---

## 3. Environment Configuration

Application behavior is dynamically controlled via OS-level environment variables (adhering to the Twelve-Factor App methodology). Secrets must **never** be committed to version control.

### 3.1 Backend Configuration Matrix

| Variable | Description | Requirement | Example/Default |
| :--- | :--- | :---: | :--- |
| `ENVIRONMENT` | Defines the runtime context (`development`, `staging`, `production`). | **Required** | `production` |
| `DATABASE_URL` | PostgreSQL connection string. | **Required** | `postgresql://user:pass@host:5432/db` |
| `OPENAI_API_KEY` | Token for LLM inference. | **Required** | `sk-proj-...` |
| `LOG_LEVEL` | Application logging verbosity. | Optional | `INFO` |
| `CORS_ORIGINS` | Comma-separated list of permitted frontend origins. | Optional | `https://aisoccopilot.com` |
| `MAX_UPLOAD_SIZE_MB` | Maximum allowed JSON upload size. | Optional | `10` |

---

## 4. Local Development Environment

The local environment is orchestrated via Docker Compose to ensure rapid bootstrapping for new developers.

### 4.1 Prerequisites
- Docker Engine & Docker Compose (v2+)
- Git

### 4.2 Local Bootstrapping Sequence
1. **Clone the repository:**
   ```bash
   git clone https://github.com/Junaidsawand/ai-soc-copilot.git
   cd ai-soc-copilot
   ```
2. **Provision Environment Variables:**
   ```bash
   cp backend/.env.example backend/.env
   # Insert your OPENAI_API_KEY into backend/.env
   ```
3. **Start the Infrastructure:**
   ```bash
   docker-compose up -d --build
   ```
4. **Execute Database Migrations:**
   ```bash
   docker-compose exec backend alembic upgrade head
   ```

### 4.3 Teardown Sequence
To cleanly shut down the local environment and preserve database volumes:
```bash
docker-compose stop
```
To destroy the environment and wipe the database:
```bash
docker-compose down -v
```

---

## 5. Production Deployment Strategy

The MVP explicitly targets managed PaaS providers (e.g., Render, Railway, or AWS App Runner) to minimize DevOps overhead.

### 5.1 CI/CD Pipeline (GitHub Actions)

```mermaid
sequenceDiagram
    participant Developer
    participant GitHub as GitHub Actions
    participant SAST as Security Scanners
    participant Test as Pytest / DB
    participant Registry as Container Registry
    participant PaaS as Production PaaS

    Developer->>GitHub: Push to `main` branch
    GitHub->>SAST: Run Bandit (Secrets Check)
    SAST-->>GitHub: Pass
    GitHub->>Test: Run Unit/Integration Tests
    Test-->>GitHub: Pass (100% Coverage)
    GitHub->>Registry: Build & Push Docker Image (Tag: Git SHA)
    Registry-->>GitHub: Image Stored
    GitHub->>PaaS: Trigger Webhook for Rolling Deployment
    PaaS->>PaaS: Pull Image & Start Container
    PaaS-->>Developer: Deployment Successful
```

### 5.2 Reverse Proxy & Networking
- **TLS Termination:** All TLS/SSL termination occurs at the PaaS Load Balancer layer. Internal traffic between the PaaS routing mesh and the backend container operates over private interfaces on HTTP.
- **HTTPS Enforcement:** The PaaS router must be configured to forcefully redirect all `HTTP 80` traffic to `HTTPS 443`.

---

## 6. Operations & Site Reliability (SRE)

### 6.1 Health Checks
The backend container exposes a lightweight health endpoint. The PaaS Load Balancer must be configured to poll this endpoint every 30 seconds.
- **Endpoint:** `GET /api/v1/health`
- **Expected Response:** `200 OK {"status": "healthy"}`
- **Action:** If the container fails 3 consecutive health checks, the orchestrator will automatically SIGTERM the container and spin up a replacement.

### 6.2 Application Logging
- The application outputs structured JSON logs to `stdout` and `stderr`.
- Logs include a `trace_id` to correlate asynchronous background AI tasks with the originating web request.
- Logs **never** contain Raw API Keys, Database Credentials, or PII.

### 6.3 Backup Strategy (RPO)
- **Database:** The managed PostgreSQL database performs daily automated point-in-time snapshots.
- **Retention:** Snapshots are retained for 7 days.
- **Recovery Point Objective (RPO):** < 24 Hours.

---

## 7. Disaster Recovery & Rollback

### 7.1 Rollback Procedure
If a deployment introduces critical failures (e.g., 500 error spikes):
1. **Halt CI/CD:** Pause GitHub Actions to prevent further merges.
2. **Revert Image Tag:** In the PaaS console, manually trigger a deployment using the previous known-good Git SHA Docker image.
3. **Database Downgrade:** If the bad deployment included a breaking schema migration, the DBA must manually execute:
   ```bash
   alembic downgrade -1
   ```
   *(Note: Migrations should be authored as backward-compatible to avoid requiring this step).*

### 7.2 Disaster Recovery (RTO)
Because all application state resides in the managed database, a total cluster failure requires:
1. Provisioning a new PaaS cluster.
2. Restoring the DB from the latest snapshot.
3. Triggering the CI/CD pipeline to deploy the containers to the new cluster.
- **Recovery Time Objective (RTO):** < 2 Hours.

---

## 8. Security Hardening

- **Non-Root Execution:** The `Dockerfile` explicitly creates a non-root `appuser`. The application will crash if execution is attempted as `root`.
- **CORS Lock Down:** In production, `CORS_ORIGINS` is strictly validated. Wildcards are forbidden.
- **Immutable Tags:** Docker images are tagged with the specific Git commit SHA. The `latest` tag is prohibited in production manifests to ensure deterministic rollbacks.
- **Secrets Management:** Secrets are injected at runtime via the PaaS secure vault. No `.env` files are copied into production containers.
