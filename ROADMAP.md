# Product Roadmap: AI SOC Copilot

The following roadmap outlines the strategic phases for developing and expanding the AI SOC Copilot platform. It progresses from a localized, single-alert analysis tool to an autonomous, enterprise-grade Security Operations ecosystem.

---

### Phase 1: Minimum Viable Product (MVP) 🟢 *(Current)*
*Establish the foundational AI investigation workflow.*
- [x] Web application interface (Flutter Web).
- [x] API Backend and Database (FastAPI & PostgreSQL).
- [x] Manual JSON File Upload for Wazuh alerts.
- [x] AI Investigation Engine (OpenAI).
- [x] MITRE ATT&CK mapping and IoC extraction.
- [x] PDF and Markdown report exporting.
- [x] Comprehensive Engineering Documentation Suite.

### Phase 2: Live Wazuh Integration 🟡 *(Planned)*
*Eliminate manual uploads via direct SIEM connectivity.*
- [ ] Implement secure Webhook API endpoint for Wazuh.
- [ ] Automate continuous alert ingestion.
- [ ] Introduce RAG (Retrieval-Augmented Generation) for querying historical alerts.

### Phase 3: Splunk Integration 🟡 *(Planned)*
*Expand market reach into enterprise SOCs.*
- [ ] Develop Splunk Universal Forwarder integration.
- [ ] Adapt JSON parsers to normalize Splunk schemas (CIM).

### Phase 4: Microsoft Sentinel Integration 🟡 *(Planned)*
*Integrate with cloud-native security environments.*
- [ ] Develop Azure Event Hub / Logic App connectors.
- [ ] Adapt JSON parsers to normalize Sentinel schemas (ASIM).

### Phase 5: Threat Hunting 🟡 *(Planned)*
*Shift from reactive analysis to proactive defense.*
- [ ] Allow analysts to query the AI to proactively search for IoCs across the historical database.
- [ ] Identify dormant advanced persistent threats (APTs).

### Phase 6: Threat Intelligence 🟡 *(Planned)*
*Enrich AI analysis with external verification.*
- [ ] Integrate APIs (e.g., VirusTotal, AlienVault OTX, GreyNoise).
- [ ] Provide real-time reputation scoring on extracted IoCs.

### Phase 7: AI Chat 🟡 *(Planned)*
*Enable interactive conversational investigations.*
- [ ] Add an interactive chat interface below investigation results.
- [ ] Allow analysts to "Ask follow-up questions" to the LLM regarding specific alerts.

### Phase 8: Multi-Tenant SaaS 🟡 *(Planned)*
*Scale the platform for enterprise B2B sales and MSSPs.*
- [ ] Implement Row-Level Security (RLS) in PostgreSQL.
- [ ] Introduce Organization and Role-Based Access Control (RBAC).
- [ ] Implement OAuth 2.0 / SSO (SAML) integrations.
