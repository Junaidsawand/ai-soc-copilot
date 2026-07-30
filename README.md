<div align="center">
  
  <img src="docs/Assets/logo_placeholder.png" alt="AI SOC Copilot Logo" width="150" height="150" />
  
  # AI SOC Copilot
  **The Intelligent Investigation Layer for Modern Security Operations**

  [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
  [![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat-square&logo=fastapi)](https://fastapi.tiangolo.com)
  [![Flutter](https://img.shields.io/badge/Flutter-%2302569B.svg?style=flat-square&logo=Flutter&logoColor=white)](https://flutter.dev)
  [![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
  [![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=flat-square&logo=openai&logoColor=white)](https://openai.com/)
  [![Status: MVP](https://img.shields.io/badge/Status-MVP-success)](#)

  <p align="center">
    <a href="#overview">Overview</a> •
    <a href="#features">Features</a> •
    <a href="#architecture">Architecture</a> •
    <a href="#tech-stack">Tech Stack</a> •
    <a href="#quick-start">Quick Start</a> •
    <a href="#documentation">Documentation</a>
  </p>
</div>

---

## 📖 Overview

**AI SOC Copilot** is an enterprise-grade cybersecurity platform designed to autonomously transform raw, complex SIEM alerts (specifically Wazuh JSON) into structured, highly readable incident investigation reports using Large Language Models (LLMs). 

Built to combat "alert fatigue," the platform drastically reduces Mean Time to Investigate (MTTI) from hours to seconds. It empowers Tier 1 SOC analysts by automating the tedious correlation of raw telemetry, extracting Indicators of Compromise (IoCs), and mapping adversary behavior directly to the **MITRE ATT&CK** framework.

**For Recruiters & Hiring Managers:**
> *This repository demonstrates full-stack software engineering proficiency, encompassing API design (FastAPI), asynchronous LLM orchestration, containerized deployment (Docker), relational database design (PostgreSQL), and modern front-end engineering (Flutter Web) within an enterprise-ready modular monolith architecture.*

---

## ✨ Features

- 🧠 **Autonomous Threat Investigation:** Upload raw Wazuh JSON alerts and receive a complete, human-readable threat narrative in seconds.
- 🛡️ **MITRE ATT&CK Mapping:** Automatically maps identified behaviors to standardized TTPs (Tactics, Techniques, and Procedures).
- 🔍 **IoC Extraction:** Deterministically extracts and structures IP addresses, file hashes, and malicious domains for immediate blocklisting.
- 📊 **Severity & Confidence Scoring:** AI assesses threat severity and assigns a confidence score to its findings to prevent hallucinated conclusions.
- 📄 **Automated Reporting:** Instantly export investigations as professional PDF or Markdown incident reports.
- ⚡ **Asynchronous Orchestration:** Non-blocking backend architecture designed to gracefully handle long-running LLM inferences without HTTP timeouts.

---

## 🏗️ Architecture

The system utilizes a **Containerized Modular Monolith** architecture, balancing robust logical boundaries with deployment simplicity.

1. **Frontend (Flutter Web):** A reactive, desktop-first SPA built for high-density SOC data visualization.
2. **Backend (FastAPI):** An asynchronous Python API handling parsing, database transactions, and business logic.
3. **AI Orchestrator:** An adapter-pattern interface communicating with external LLM providers (e.g., OpenAI) while strictly validating outputs via Pydantic schemas.
4. **Persistence (PostgreSQL):** A highly normalized relational database storing immutable alert telemetry and investigation ledgers.

---

## 💻 Tech Stack

| Domain | Technology |
|---|---|
| **Frontend** | Flutter, Dart, Provider/Riverpod |
| **Backend** | Python 3.11+, FastAPI, Uvicorn |
| **Database** | PostgreSQL 16+, SQLAlchemy, Alembic |
| **AI / NLP** | OpenAI API, LangChain (Future) |
| **Infrastructure** | Docker, Docker Compose, GitHub Actions |

---

## 📸 Screenshots

*(Replace placeholders with actual UI screenshots once deployed)*

| Dashboard | Investigation Results |
| :---: | :---: |
| <img src="docs/Assets/screenshot_dashboard.png" alt="Dashboard View" width="400"/> | <img src="docs/Assets/screenshot_results.png" alt="Results View" width="400"/> |

---

## 🚀 Quick Start & Installation

### Prerequisites
- Docker and Docker Compose installed.
- An active OpenAI API Key.

### Local Development Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Junaidsawand/ai-soc-copilot.git
   cd ai-soc-copilot
   ```

2. **Configure Environment Variables:**
   ```bash
   cp backend/.env.example backend/.env
   # Edit backend/.env and insert your OPENAI_API_KEY
   ```

3. **Spin up the Backend and Database:**
   ```bash
   docker-compose up -d --build
   ```
   *The FastAPI server is now running at `http://localhost:8000/api/v1`*<br>
   *Interactive API Docs (Swagger): `http://localhost:8000/docs`*

4. **Run Database Migrations:**
   ```bash
   docker-compose exec backend alembic upgrade head
   ```

5. **Start the Frontend:**
   ```bash
   cd frontend
   flutter pub get
   flutter run -d chrome
   ```

---

## 📂 Folder Structure

```text
ai-soc-copilot/
├── backend/                  # FastAPI Application (API, Services, AI Orchestrator)
├── frontend/                 # Flutter Web SPA (UI, State Management, API Clients)
├── docs/                     # Comprehensive Architecture & System Documentation
├── docker-compose.yml        # Local infrastructure orchestration
└── README.md                 # Project root documentation
```

---

## 📚 Documentation Links

For deep dives into the system design, please review the extensive engineering documentation:
- 📖 [Product Requirements Document (PRD)](docs/01-PRD/PRD.md)
- 🏗️ [System Architecture](docs/02-System-Architecture/System_Architecture.md)
- 🗄️ [Database Design](docs/03-Database-Design/Database_Design.md)
- 🔌 [API Specification](docs/04-API-Specification/API_Specification.md)
- 🧠 [AI Design & Prompt Strategy](docs/05-AI-Design/AI_Design.md)
- 🎨 [UI/UX Specification](docs/06-UI-UX/UI_UX_Design.md)
- 🔒 [Security Architecture](docs/09-Security-Architecture/Security_Architecture.md)
- 👨‍💻 [Developer Onboarding Guide](docs/10-Developer-Guide/Developer_Guide.md)

---

## 🗺️ Roadmap

- [x] **Phase 1 (MVP):** Single-file JSON upload, OpenAI analysis, PDF export.
- [ ] **Phase 2:** Retrieval-Augmented Generation (RAG) using historical alert context.
- [ ] **Phase 3:** Live WebSocket integration with Wazuh Server.
- [ ] **Phase 4:** Multi-Tenant Role-Based Access Control (RBAC).

---

## 🤝 Contributing

Contributions are what make the open-source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feat/AmazingFeature`)
3. Commit your Changes (`git commit -m 'feat(module): add some AmazingFeature'`)
4. Push to the Branch (`git push origin feat/AmazingFeature`)
5. Open a Pull Request

*Please refer to our [Developer Guide](docs/10-Developer-Guide/Developer_Guide.md) for strict coding standards and Git conventions.*

---

## 📝 License

Distributed under the MIT License. See `LICENSE` for more information.

---

## 👨‍💻 Author

**Junaid Sawand**
- LinkedIn: [Your Profile](https://linkedin.com/in/yourprofile)
- Portfolio: [Your Website](https://yourportfolio.com)
- GitHub: [@Junaidsawand](https://github.com/Junaidsawand)

---

## 🙏 Acknowledgements

- [Wazuh](https://wazuh.com/) for their incredible open-source SIEM/XDR platform.
- [FastAPI](https://fastapi.tiangolo.com/) for the high-performance Python framework.
- [MITRE ATT&CK](https://attack.mitre.org/) for standardizing global threat knowledge.
