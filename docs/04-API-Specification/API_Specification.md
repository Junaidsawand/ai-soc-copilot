# API Specification: AI SOC Copilot

## 1. Executive Summary
This document specifies the RESTful API contract for the AI SOC Copilot platform. It serves as the definitive reference for frontend integration and backend implementation.

The API is engineered to be **OpenAPI 3.0 compatible** (auto-generated via FastAPI) and strictly adheres to REST principles, ensuring statelessness, uniform response envelopes, and deterministic error handling.

---

## 2. Global API Conventions

### 2.1 Base URLs & Versioning
The API enforces URI-based versioning to guarantee backward compatibility for clients as the platform evolves.

| Environment | Base URL |
|---|---|
| **Local Development** | `http://localhost:8000/api/v1` |
| **Production** | `https://api.aisoccopilot.com/api/v1` |

### 2.2 Standard Envelopes
To simplify frontend parsing and type generation (e.g., Dart/Flutter Data Classes), every API response is wrapped in a consistent JSON envelope.

**Success Response Envelope (`HTTP 2xx`)**
```json
{
  "success": true,
  "data": { ... }, 
  "timestamp": "2026-07-21T12:30:00Z"
}
```
*(Note: `data` can be an object or an array depending on the endpoint).*

**Error Response Envelope (`HTTP 4xx`, `HTTP 5xx`)**
```json
{
  "success": false,
  "error": {
    "code": "INVALID_SCHEMA",
    "message": "The JSON structure does not conform to the Wazuh alert format.",
    "details": [
      {"loc": ["body", "rule", "level"], "msg": "field required", "type": "value_error.missing"}
    ]
  },
  "timestamp": "2026-07-21T12:31:00Z"
}
```

### 2.3 Authentication Readiness (Future Phase)
While the MVP defers active user authentication, the API is structured to adopt **OAuth2 with JWT Bearer Tokens**. 
- Clients should be prepared to inject `Authorization: Bearer <token>` into the HTTP headers.
- Endpoints will return `401 Unauthorized` for missing tokens and `403 Forbidden` for RBAC violations.

---

## 3. Endpoints Directory

### 3.1 Investigations
Manages the core lifecycle of security alerts and AI analysis.

#### 3.1.1 Upload & Initiate Investigation
Uploads a Wazuh JSON alert and triggers asynchronous AI processing.
- **Method:** `POST`
- **Path:** `/investigations`
- **Content-Type:** `multipart/form-data`
- **Request Body:**
  - `file`: (Binary) The `.json` Wazuh alert payload (Max 10MB).
- **Responses:**
  - `202 Accepted`: File validated, processing queued.
  ```json
  {
    "success": true,
    "data": { "investigation_id": "uuid-v4", "status": "pending" },
    "timestamp": "..."
  }
  ```
  - `415 Unsupported Media Type`: File is not `.json`.
  - `413 Payload Too Large`: Exceeds 10MB limit.
  - `422 Unprocessable Entity`: Invalid JSON syntax.

#### 3.1.2 Get Investigation Status & Results
Retrieves the real-time status or final AI analysis of a specific investigation. Frontend clients should poll this endpoint (e.g., every 2s) while status is `pending` or `analyzing`.
- **Method:** `GET`
- **Path:** `/investigations/{investigation_id}`
- **Responses:**
  - `200 OK`:
  ```json
  {
    "success": true,
    "data": {
      "id": "uuid-v4",
      "status": "completed",
      "severity": "high",
      "executive_summary": "...",
      "mitre_mapping": [...],
      "iocs": [...],
      "recommendations": [...]
    },
    "timestamp": "..."
  }
  ```
  - `404 Not Found`: Invalid ID.

#### 3.1.3 List Historical Investigations
Retrieves a paginated list of past investigations.
- **Method:** `GET`
- **Path:** `/investigations`
- **Query Parameters:**
  - `page` (int, default=1)
  - `size` (int, default=20, max=100)
  - `severity` (string, optional: `critical`, `high`, `medium`, `low`)
  - `sort` (string, default=`-created_at`)
- **Responses:**
  - `200 OK`:
  ```json
  {
    "success": true,
    "data": {
      "items": [ { "id": "...", "severity": "high", "created_at": "..." } ],
      "total_items": 142,
      "page": 1,
      "pages": 8
    },
    "timestamp": "..."
  }
  ```

#### 3.1.4 Delete Investigation
Removes an investigation from the system (Soft delete).
- **Method:** `DELETE`
- **Path:** `/investigations/{investigation_id}`
- **Responses:**
  - `204 No Content`: Successful deletion.
  - `404 Not Found`: Invalid ID.

---

### 3.2 Reporting
*Design Note: To strictly adhere to REST principles, reporting is modeled as a sub-resource of an investigation, rather than an isolated RPC endpoint.*

#### 3.2.1 Export Investigation Report
Dynamically generates and returns a physical report file for an investigation.
- **Method:** `GET`
- **Path:** `/investigations/{investigation_id}/report`
- **Query Parameters:**
  - `format` (string, required): `pdf` or `markdown`
- **Responses:**
  - `200 OK`: Returns the raw binary stream.
    - `Content-Type: application/pdf` OR `text/markdown`
    - `Content-Disposition: attachment; filename="report-uuid.pdf"`
  - `404 Not Found`: Investigation does not exist.
  - `400 Bad Request`: If the investigation status is not `completed`.

#### 3.2.2 Regenerate AI Analysis (RPC/Action)
Instructs the AI to re-evaluate the original alert payload. 
- **Method:** `POST`
- **Path:** `/investigations/{investigation_id}/actions/reanalyze`
- **Responses:**
  - `202 Accepted`: Reanalysis queued. The frontend should resume polling `GET /investigations/{id}`.

---

### 3.3 System Health

#### 3.3.1 Health Probe
- **Method:** `GET`
- **Path:** `/health`
- **Responses:**
  - `200 OK`:
  ```json
  {
    "status": "healthy",
    "dependencies": {
      "database": "connected",
      "openai_api": "available"
    },
    "version": "1.0.0"
  }
  ```
  - `503 Service Unavailable`: If DB connection fails.

---

## 4. HTTP Status & Error Code Catalog

### 4.1 Standard HTTP Codes
| Code | Reason | Typical Use Case |
|---|---|---|
| `200` | OK | Standard GET requests, paginated lists. |
| `202` | Accepted | Async task queued (File Upload, Reanalyze). |
| `204` | No Content | Successful DELETE. |
| `400` | Bad Request | Business logic error (e.g., generating report for incomplete investigation). |
| `401` | Unauthorized | Missing or invalid auth token (Future). |
| `403` | Forbidden | Insufficient RBAC permissions (Future). |
| `404` | Not Found | UUID does not exist in DB. |
| `413` | Payload Too Large| Upload exceeds 10MB limits. |
| `415` | Unsupported Media| File is not `.json`. |
| `422` | Unprocessable | Pydantic schema validation failed (e.g., invalid email, missing required JSON keys). |
| `429` | Too Many Requests| API Rate limit triggered. |
| `500` | Internal Error | Unhandled backend exception. |
| `502` | Bad Gateway | OpenAI API is unreachable or returned invalid data. |
| `503` | Unavailable | Database offline. |

### 4.2 Application Error Codes (`error.code`)
Frontend clients should map UI feedback based on the internal `error.code` string rather than just the HTTP status.

| Code | Description | User Facing Action |
|---|---|---|
| `INVALID_SCHEMA` | Uploaded JSON is not a Wazuh alert. | "Please upload a valid Wazuh JSON alert." |
| `FILE_TOO_LARGE` | Payload > 10MB. | "File exceeds 10MB limit." |
| `AI_TIMEOUT` | OpenAI took too long to respond. | "Analysis timed out. Please try again." |
| `AI_PROVIDER_ERROR`| OpenAI returned a 5xx. | "AI service temporarily degraded." |
| `DB_TRANSACTION_FAILED`| Persistence layer failed. | "System error saving investigation." |

---

## 5. OpenAPI & Validation
- **Interactive Documentation:** Upon backend launch, full interactive documentation is automatically hosted at `/docs` (Swagger UI) and `/redoc`.
- **Frontend Readiness:** The API is 100% ready for automated client generation. Flutter developers can consume the OpenAPI `openapi.json` spec using tools like `openapi-generator-cli` or `chopper` to automatically build strongly-typed Dart data models and API services, guaranteeing frontend-backend alignment.
