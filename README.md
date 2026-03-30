# [Project Name: AI-Powered Delivery Health & Operations Intelligence System]

![AI-Powered Delivery Health & Operations Intelligence System](crebos%20banner.png)

> A production-grade **n8n orchestration** system designed to monitor delivery health across multiple concurrent projects. By replacing gut-feel decisions with objective **AI diagnostics** and two-way **ClickUp synchronization**, this system provides a reliable, data-driven escalation path.

---

### **System Highlights (From Production Pilot @ Crebos.online)**

* ✅ **Objective Health Scoring:** Deterministic RAG (Red/Yellow/Green) scoring across all live projects.
* ✅ **AI-Generated Diagnostics:** Intelligent issue identification and prioritized recommendations powered by Gemini AI.
* ✅ **Anti-Spam Controls:** Aggregation logic prevents alert fatigue (eliminating the "36+ duplicate emails" problem).
* ✅ **Fail-Safe Manual Review:** Zero silent failures; tasks with unclear data or malformed AI are automatically routed for human judgement.
* ✅ **Strict Input Validation:** Modular workflow condition checks and type-checking ensure architecture stability.
* ✅ **Two-Way Sync:** Verified updates are written directly back to ClickUp, the single source of truth.

---

### **Architecture Overview**

The pipeline consists of **25+ n8n nodes**, organized into three core operational phases:

1.  **Phase I: Data Prep & Task Update:**ClickUp Get tasks -> Data Normalization -> Scoring Engine -> Gemini AI Enrichment -> AI Output Validation -> Loop-based ClickUp Update.
2.  **Phase II: Verification & Routing:**Fetch Updated State -> Manual Review Detection -> Aggregated Alerts (Anti-Spam Fix).
3.  **Phase III: Notifications & Completion:**Alert Routing (Status Alert vs. Manual Review) -> Gmail Notification -> Manual Task Creation -> End-to-End Log.

---
