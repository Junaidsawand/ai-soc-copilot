# Quality Assurance & Testing Strategy: AI SOC Copilot

## 1. Executive Summary
This document formalizes the enterprise testing strategy for the AI SOC Copilot MVP. The primary objective is to guarantee the reliability, security, and deterministic behavior of the platform before it is deployed to production. 

Given the integration of non-deterministic Large Language Models (LLMs) into the core workflow, this QA strategy places heavy emphasis on strict schema validation, AI behavioral testing, and comprehensive automated test coverage across the entire stack.

---

## 2. Test Automation Pyramid

To ensure rapid feedback loops and maintainable code, the QA strategy adheres to the testing pyramid: maximizing unit test coverage, utilizing integration tests for component boundaries, and reserving E2E tests for critical user journeys.

### 2.1 Unit Testing
**Objective:** Validate individual functions, methods, and classes in isolation.
- **Backend (FastAPI):** Utilizing `pytest`.
  - **Targets:** Wazuh JSON parser logic, Pydantic schema validation, LLM Prompt Builder logic.
  - **Mocking:** The database ORM (SQLAlchemy) and the `AI Orchestrator` provider interface must be heavily mocked to ensure tests run rapidly and deterministically without network calls.
- **Frontend (Flutter):** Utilizing `flutter test`.
  - **Targets:** State management logic, JSON serialization/deserialization models, and core UI widget rendering.

### 2.2 Integration Testing
**Objective:** Verify that independent modules operate correctly when connected.
- **API Integration:** Using `pytest` and `TestClient` (FastAPI).
  - Verify HTTP status codes, correct routing, and payload validation (e.g., uploading an invalid `.txt` file instead of `.json`).
- **Database Integration:** Utilizing an ephemeral test database (e.g., PostgreSQL inside a Docker container) to verify CRUD operations, schema migrations (Alembic), and foreign key constraints without polluting development databases.

### 2.3 End-to-End (E2E) Testing
**Objective:** Validate the complete system workflow from the user's perspective, running against a fully deployed staging environment.
- **Tooling:** Cypress or Playwright (for web UI).
- **Core Scenarios (The Happy Path):**
  1. User navigates to Dashboard.
  2. User uploads a mock Wazuh alert.
  3. System displays loading states.
  4. System transitions to Investigation Results view.
  5. User exports the report as a PDF.
- *Note: To avoid incurring LLM API costs during automated CI E2E runs, the AI Orchestrator must be configurable to return a static, pre-recorded JSON response.*

---

## 3. Specialized Testing Domains

### 3.1 AI Output Validation & Testing
Testing non-deterministic LLMs requires specialized strategies to ensure the system doesn't break due to unexpected output.
- **Schema Conformance Tests:** Assert that the backend's Pydantic validators successfully reject malformed JSON from the LLM, and that the retry mechanism correctly triggers.
- **Golden Dataset Testing:** A suite of known, diverse Wazuh alerts (the "Golden Dataset") is processed through the LLM. Engineers manually verify that the AI consistently identifies the correct MITRE tactic and severity, minimizing regression as prompt templates are adjusted.
- **Hallucination Probes:** Feeding the system alerts with intentionally sparse or corrupted data to verify the AI gracefully degrades and reports "Low Confidence" rather than inventing threat narratives.

### 3.2 Performance & Load Testing
**Objective:** Ensure the system handles concurrent usage and large payloads gracefully.
- **Tooling:** K6 or Locust.
- **Targets:**
  - **Payload Stress:** Uploading JSON files right at the 10MB limit to verify parser memory efficiency and API timeout resilience.
  - **Concurrency:** Simulating 50 concurrent investigation requests to ensure the FastAPI background task queue and SQLAlchemy connection pool do not exhaust resources or deadlock.
- **SLOs:** API must accept the upload and return a `202 Accepted` within 500ms, regardless of payload size.

### 3.3 Security Testing
**Objective:** Prevent vulnerabilities and ensure infrastructure resilience.
- **Static Application Security Testing (SAST):** Utilizing tools like `Bandit` (Python) and SonarQube in the CI pipeline to catch hardcoded secrets or insecure configurations.
- **Dependency Scanning:** Automated auditing of `requirements.txt` (Python) and `pubspec.yaml` (Flutter) for known CVEs (e.g., via Dependabot or Snyk).
- **Upload Sanitization Testing:** Explicitly attempting to upload malicious payloads (e.g., recursive JSON bombs, excessively deep nested JSON) to verify the parser safely rejects them without crashing (Denial of Service mitigation).

---

## 4. Regression & Continuous Integration (CI)

### 4.1 CI/CD Pipeline Gates
Code cannot be merged into the `main` branch unless it passes the following automated QA gates in GitHub Actions (or equivalent):
1. **Linting & Formatting:** Code adheres to PEP-8 (Python/Black) and Dart analyzer standards.
2. **Unit Tests:** 100% pass rate.
3. **Integration Tests:** 100% pass rate.
4. **Test Coverage:** Code coverage remains above 80%.

### 4.2 Regression Testing
Before any production release, an automated regression suite is triggered. This ensures that new features (e.g., a new UI layout) do not break core legacy functionality (e.g., PDF generation).

---

## 5. User Acceptance Testing (UAT)

**Objective:** Validate that the built software actually solves the user's business problem.
- **Participants:** Security engineering peers, cybersecurity students, and internal stakeholders acting as Tier 1 SOC analysts.
- **UAT Criteria:**
  - The generated AI summary is technically accurate and coherent.
  - The UI does not induce cognitive fatigue (i.e., information is well-structured).
  - The workflow is intuitively navigable without relying on extensive training manuals.
- **Sign-Off:** Formal sign-off from the Product Owner is required following successful UAT before the MVP can be deployed to production.
