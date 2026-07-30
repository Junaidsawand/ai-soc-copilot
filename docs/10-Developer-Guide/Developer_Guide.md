# Developer Guide: AI SOC Copilot

## 1. Welcome to the Team
Welcome to the AI SOC Copilot engineering team. This document serves as your definitive onboarding guide. It covers everything from bootstrapping your local environment to our rigorous Git contribution standards. 

Our stack consists of a **FastAPI (Python 3.11+)** backend and a **Flutter Web (Dart)** frontend, utilizing a **PostgreSQL** database.

---

## 2. Project Architecture & Folder Structure

The project is structured as a **Modular Monolith** housed within a monorepo.

```text
ai-soc-copilot/
├── backend/                  # FastAPI Application
│   ├── app/
│   │   ├── api/              # HTTP routers and controllers
│   │   ├── core/             # Application config and security
│   │   ├── domain/           # Business logic (Parser, AI Orchestrator)
│   │   ├── models/           # SQLAlchemy ORM definitions
│   │   ├── schemas/          # Pydantic validation models
│   │   └── services/         # Orchestration and external integrations
│   ├── tests/                # Pytest unit and integration tests
│   ├── alembic/              # Database migration scripts
│   ├── requirements.txt      # Python dependencies
│   └── Dockerfile            # Container definition
├── frontend/                 # Flutter Web Application
│   ├── lib/
│   │   ├── core/             # Themes, constants, routing
│   │   ├── models/           # Dart data models (generated)
│   │   ├── screens/          # Primary UI views (Dashboard, Results)
│   │   ├── services/         # API HTTP clients
│   │   └── widgets/          # Reusable UI components
│   ├── test/                 # Flutter test suite
│   └── pubspec.yaml          # Dart dependencies
├── docs/                     # Architectural & Engineering documentation
└── docker-compose.yml        # Local development orchestration
```

---

## 3. Local Environment Setup

### 3.1 Prerequisites
Ensure the following tools are installed on your workstation:
- **Docker & Docker Compose:** For running the local database and backend containers.
- **Python 3.11+:** For backend development.
- **Flutter SDK (Stable channel):** For frontend development.
- **Git:** Version control.
- **IDE:** VS Code or IntelliJ/PyCharm (with Python and Flutter plugins installed).

### 3.2 Environment Variables
Never commit secrets. Copy the example environment files to create your local overrides:
```bash
# In the backend directory
cp .env.example .env
```
Update `.env` with your personal `OPENAI_API_KEY`. The local database URL will default to the Docker container (`postgresql://user:pass@localhost:5432/aisoc`).

### 3.3 Bootstrapping the Backend
The easiest way to run the backend and database is via Docker Compose:
```bash
# From the project root
docker-compose up -d --build
```
This command spins up:
1. **PostgreSQL Container:** Available on port `5432`.
2. **FastAPI Container:** API available at `http://localhost:8000/api/v1`.
3. **Swagger Docs:** Interactive API docs at `http://localhost:8000/docs`.

To run database migrations locally:
```bash
docker-compose exec backend alembic upgrade head
```

### 3.4 Bootstrapping the Frontend
To run the Flutter web application locally:
```bash
cd frontend
flutter pub get
flutter run -d chrome
```
The application will be served at `http://localhost:XXXX` (Flutter provides the dynamic port in the console).

---

## 4. Coding Standards

### 4.1 Backend (Python / FastAPI)
- **Typing:** Strict Python type hints are mandatory (`def process_alert(alert_id: UUID) -> Dict[str, Any]:`).
- **Formatting:** We use **Black** for code formatting and **Ruff** for linting. Ensure your IDE is configured to format-on-save.
- **Validation:** All incoming API payloads must be validated using **Pydantic** schemas. Do not use raw dictionaries.
- **Database:** Always use the SQLAlchemy async ORM. Never construct raw SQL queries via f-strings (prevents SQL injection).

### 4.2 Frontend (Dart / Flutter)
- **State Management:** Use the designated state management provider (e.g., Provider or Riverpod). Avoid heavy logic inside stateful widgets.
- **Widget Granularity:** Break large UI screens into small, reusable, stateless widgets.
- **Linting:** We strictly enforce the `flutter_lints` package. No warnings should be present in your PRs.

---

## 5. Git Conventions & Contribution Workflow

We utilize a simplified **Trunk-Based Development** workflow with Pull Requests (PRs).

### 5.1 Branch Naming Convention
Branches must be descriptive and categorized:
- `feat/add-pdf-export`
- `fix/parser-crash-on-null-ip`
- `chore/update-dependencies`
- `docs/api-spec-update`

### 5.2 Commit Messages (Conventional Commits)
Commit messages must follow the Conventional Commits specification to automate changelog generation.
- **Format:** `<type>(<scope>): <description>`
- **Examples:**
  - `feat(api): add endpoint for pdf export`
  - `fix(parser): gracefully handle missing agent name`
  - `docs(readme): update local setup instructions`

### 5.3 The Pull Request (PR) Lifecycle
1. **Branch off `main`**: Ensure your local `main` is up to date.
2. **Commit small, cohesive changes**: Do not bundle 3 features into a single PR.
3. **Write Tests**: Ensure unit tests cover your new logic. Run them locally before pushing.
4. **Open a PR against `main`**: Fill out the PR template completely.
5. **CI/CD Checks**: GitHub Actions will automatically run linting and tests. Your PR **cannot** be merged if CI fails.
6. **Code Review**: At least one peer review approval is required.
7. **Squash and Merge**: Use the "Squash and Merge" feature to keep the `main` branch history clean.

---

## 6. Testing Expectations

- **Unit Tests:** You must write tests for new business logic (e.g., a new parser rule).
- **Running Backend Tests:**
  ```bash
  docker-compose exec backend pytest
  ```
- **Running Frontend Tests:**
  ```bash
  cd frontend
  flutter test
  ```

---

## 7. Troubleshooting Common Issues

### Issue: "Alembic cannot connect to the database"
- **Cause:** The PostgreSQL Docker container is not running or hasn't fully initialized.
- **Fix:** Run `docker ps` to ensure the `db` container is up. Check logs with `docker-compose logs db`.

### Issue: "Pydantic ValidationError on JSON Upload"
- **Cause:** The test JSON you are uploading is missing required fields defined in our `AlertSchema`.
- **Fix:** Check the Swagger UI (`http://localhost:8000/docs`) to see the exact required JSON structure, or review the error output which specifies the missing keys.

### Issue: "Flutter Web fails to build due to CORS"
- **Cause:** When running Flutter Web locally, browsers enforce CORS, which can block local API requests.
- **Fix:** Ensure the FastAPI backend has `http://localhost:*` in its allowed CORS origins array within `.env.development`.

### Issue: "OpenAI API returns 401 Unauthorized"
- **Cause:** Missing or invalid API key.
- **Fix:** Verify your `.env` file contains a valid `OPENAI_API_KEY` and restart the backend container.
