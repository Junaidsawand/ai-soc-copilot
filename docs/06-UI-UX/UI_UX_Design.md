# UI/UX Design Specification: AI SOC Copilot

## 1. Executive Summary
This document establishes the UI/UX specifications for the AI SOC Copilot MVP. The design philosophy is engineered specifically for Security Operations Center (SOC) environments, where cognitive load reduction, high data density, and deterministic feedback are critical.

The platform is designed as a **Desktop-First Web Application** using Flutter Web, optimizing for the large, multi-monitor arrays typical in enterprise security operations.

---

## 2. Core UX Principles
- **Simplicity Over Complexity:** Workflows must be direct and linear. Avoid deeply nested menus or hidden configuration panels.
- **Evidence-Based Transparency:** Always display original Wazuh alert telemetry alongside AI-generated interpretations to maintain analytical trust.
- **Progressive Disclosure:** Surface executive summaries and severity scores immediately. Reveal deep technical telemetry (e.g., raw JSON, MITRE mappings) only upon explicit user interaction.
- **Deterministic Feedback:** Avoid arbitrary percentage loaders. Users must always know the precise state of backend processing.

---

## 3. User Flow Architecture

The MVP strictly isolates a single, high-value primary user journey.

```mermaid
flowchart TD
    A[Dashboard Landing] --> B[New Investigation Upload]
    B --> C[Stateful Processing UI]
    C --> D[Investigation Results View]
    
    D -->|Download PDF/MD| E[Export Artifact]
    D -->|Navigate| F[Investigation Ledger (History)]
    
    F -->|Query / Filter| F
    F -->|Select Record| G[Historical Results View]
```

### 3.1 Primary Navigation Model
The application utilizes a persistent, top-level navigation bar to maximize vertical screen real estate for data tables.
- **Dashboard (`/`)**
- **New Investigation (`/investigate/new`)**
- **History (`/history`)**
- **About (`/about`)**

---

## 4. Component Hierarchy & Wireframe Specifications

### 4.1 Screen: Dashboard
**Objective:** Provide a high-level operational overview and direct access to primary workflows.
- **Hero Area:** "Start New Investigation" Primary Call-To-Action (CTA).
- **Widgets Layer:** 
  - Total Investigations Processed (KPI)
  - Critical/High Severity Detections (KPI)
  - Average AI Processing Latency (KPI)
- **Data Layer:** A truncated data grid showing the 5 most recent investigations.

### 4.2 Screen: New Investigation (Upload)
**Objective:** Secure ingestion point for Wazuh alerts.
- **Component:** Massive, highly visible Dropzone supporting OS drag-and-drop.
- **Helper Text:** Explicitly states `.json` format and `10 MB` limits.
- **Action Buttons:** `[ Submit ]` and `[ Cancel ]`.

### 4.3 Screen: Investigation Results (Core View)
**Objective:** The focal point of the application, rendering AI threat intelligence.
- **Header:** Investigation ID, Processing Timestamp, Export Controls (PDF/MD).
- **Left Column (Summary & TTPs):**
  - Severity Badge (Color-coded: Red=Critical, Orange=High, Yellow=Medium, Blue=Low).
  - Confidence Score.
  - Executive Summary.
  - Extracted Indicators of Compromise (IoCs).
  - MITRE ATT&CK Matrix references.
- **Right Column (Narrative & Evidence):**
  - AI Attack Narrative.
  - Recommended Remediation Steps.
  - *Collapsible Accordion:* Raw Wazuh Alert JSON (for forensic verification).

### 4.4 Global UI Components
Implemented as highly reusable Flutter Widgets:
- **Toasts/Snackbars:** Ephemeral, non-blocking feedback (e.g., "Report Downloaded").
- **Error Banners:** Persistent, blocking alerts (e.g., "AI Provider Offline").
- **Action Modals:** Required for destructive actions (e.g., "Delete Investigation").

---

## 5. State Management UX

### 5.1 Stateful Loading (Progress Indicators)
During synchronous backend operations (e.g., LLM inference), the UI must gracefully degrade:
- **Lock Controls:** Disable all primary action buttons to prevent duplicate POST requests.
- **Deterministic Stages:** Replace infinite spinners with step-based feedback:
  1. `[✓] Upload Complete`
  2. `[✓] Validation Passed`
  3. `[⏳] AI Analyzing Telemetry...`
  4. `[○] Generating Report`

### 5.2 Error States
Error states must be constructive, guiding the user toward resolution.
- **Validation Errors:** Highlight the dropzone in red, rendering: *"Invalid file format. Please ensure the payload is a valid Wazuh JSON schema."*
- **Service Outages:** Display a global error banner: *"AI Service is currently unreachable. Please try again in a few moments."*

### 5.3 Empty States
Empty states must include educational messaging or CTAs, not blank screens.
- **Dashboard:** *"No investigations yet. Upload your first Wazuh alert to begin."*
- **History Table:** *"No investigations match your filter criteria."*

---

## 6. Accessibility (a11y) & Responsiveness

### 6.1 Accessibility Standards
- **Contrast Ratios:** The color palette (Security Dark Mode or High-Contrast Light Mode) must adhere to **WCAG AA** standards (minimum 4.5:1 contrast for text).
- **Keyboard Navigation:** Full tab-traversal support for all buttons, links, and form elements.
- **Focus Indicators:** Explicit, high-contrast outlines for active elements.
- **ARIA Labeling:** Screen-reader compatible text alternatives for all icon-only buttons (e.g., the Export icon).

### 6.2 Responsive Degradation Strategy
The MVP is optimized for **Desktop (`≥ 1280px`)**.
- **Tablet (`768px - 1279px`):** The UI gracefully compresses. Side-by-side columns (like the Results View) stack vertically. Data grids introduce horizontal scrolling.
- **Mobile (`< 768px`):** Basic rendering is supported via vertical stacking, though mobile is explicitly out-of-scope as a primary target for the MVP.

---

## 7. Design System & Aesthetics
- **Visual Identity:** Authoritative, modern, and trustworthy. Avoid playful or overly vibrant consumer palettes.
- **Typography:** Utilize highly legible, monospaced fonts for technical data (IPs, hashes, JSON) and scalable sans-serif fonts for narrative text.
- **Color Semantics:** Strictly reserve Red, Orange, and Green for systemic status indicators (Severity, Errors, Success). Do not use these colors for arbitrary decorative elements.
