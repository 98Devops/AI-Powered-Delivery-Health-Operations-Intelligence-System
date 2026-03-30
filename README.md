# AI-Powered Delivery Health & Operations Intelligence System]

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

## This repo contains the System Documenation and the n8n workflow in json                                                                                           # **Daily Delivery Health Check System – Full Technical Documentation**

**AI & Workflow Automation Pilot – Crebos Online Solutions**

## **1. Purpose of the System**

The Daily Delivery Health Check workflow automates monitoring of ongoing projects inside ClickUp and provides:

- **Objective scoring** (RED, YELLOW, GREEN)
- **AI-generated key issues and recommendations**
- **ClickUp task updates**
- **Escalation routing** for risky or unclear projects
- **Consolidated alerts** (so the ops team doesn’t get spammed)
- **Fail-safe manual review loop** for malformed AI or ambiguous statuses

This system is built to run **daily**, fully hands-off.

---

# **2. Architecture Overview**

The workflow is built on **three synchronized phases**, each with clear responsibility separation:

---

## **Phase I: Data Preparation & Task Update (Core Processing)**

Tasks are fetched, normalized, scored, enriched by AI, validated, and updated before any alerts are sent.

### Key Operations in this Phase:

1. **ClickUp: Get Tasks**
    - Retrieves all project tasks from the target list.
    - Includes custom fields, blocker descriptions, progress, deadlines.
2. **Code: Normalize Project Data**
    - Converts all inputs into a clean, predictable schema.
    - Converts percentages to integers.
    - Ensures strings like `"null"` become actual null.
    - Computes **deadline pressure** (days left).
    - Eliminates undefined values that break AI.
3. **Code: Scoring Engine**
    - Implements deterministic cascaded logic:
        - If any Red rule fires → Status = Red.
        - Else if any Yellow rule fires → Status = Yellow.
        - Else → Green.
    - Ensures consistency regardless of inconsistent ClickUp fields.
4. **Gemini AI Enrichment**
    - Produces:
        - `aiKeyIssues`
        - `aiRecommendation`
        - `aiScoreValidation` (LLM cross-check)
    - Auto-retries 3 times.
    - Strict JSON output enforcement.
5. **Code: Validate AI Output**
    - If any of the required fields are missing, empty, malformed, or contain hallucination artifacts → route to **Manual Review**.
    - Ensures workflow doesn’t break when AI misbehaves.
6. **Loop + ClickUp Update (Batch)**
    - Each task gets updated with:
        - Score
        - Status Color
        - AI Valid (boolean)
        - Recommendation
        - Key Issues
        - Last Checked timestamp
7. **Wait Node (3 seconds)**
    - Allows ClickUp backend to commit all updates before next phase.
    - Fixes inconsistencies where updates appear in API but not UI instantly.

---

## **Phase II: Verification & Routing Logic**

This phase classifies tasks into two types:

### **1. Status Alerts**

Tasks that:

- have valid AI,
- and status is **Red** or **Yellow**.

### **2. Manual Reviews**

Tasks that:

- have invalid AI output,
- or missing core fields,
- or suspiciously inconsistent data.

---

### Nodes in This Phase:

1. **ClickUp Get Tasks (Fetch Updated State)**
    - Pulls fresh state post-update.
    - Confirms fields are committed.
2. **Code: Manual Review Detection**
    
    For each task, generates:
    
    ```json
    {
      "taskId": "...",
      "taskName": "...",
      "score": 25,
      "status": "Red",
      "blockers": "...",
      "aiValid": true,
      "notificationType": "STATUS_ALERT" | "MANUAL_REVIEW"
    }
    
    ```
    
3. **Code: Aggregate Alerts (Anti-Spam Fix)**
    - This is the **core architectural fix**.
    - Instead of n8n emitting 30 items (one per task):
        - It emits **one single item** containing two arrays:
            - `statusAlerts: []`
            - `manualReviews: []`
        - Also includes:
            - `statusAlertCount`
            - `manualReviewCount`

**This change eliminated the 36+ spam emails problem.**

---

## **Phase III: Notification, Manual Task Creation & Completion**

This is the output/communication layer.

### Nodes:

1. **IF: Status Alerts Exist?**
    - Checks `statusAlertCount > 0`
    - If true → Generate email.
2. **IF: Manual Reviews Exist?**
    - Checks `manualReviewCount > 0`
    - If true:
        - Creates a new **Manual Review ClickUp Task**
        - Sends manual review email summary
3. **Gmail: Status Alert Email**
    
    Format includes:
    
    - Project name
    - Score
    - Blockers
    - Key Issues (AI)
    - AI Recommendations (AI)
    - Link to task
4. **Gmail: Manual Review Email**
    
    Summary of all projects requiring human evaluation.
    
5. **Merge Node (Wait for All Inputs)**
    - Ensures completion only logs after all alert tasks are sent.
6. **Final Comment / Final Log (Optional)**
    - Marks the run complete.

# **3. Key Engineering Decisions & Trade-Offs**

| Decision | Why it matters | Trade-off |
| --- | --- | --- |
| **Aggregation before routing** | Prevented 36 duplicate emails. Clean architecture. | Required rethinking of pipeline. |
| **Formally validated AI output** | AI never breaks downstream nodes. | Additional code node. |
| **Two-pass ClickUp fetch** | Guarantees accurate verification after updates. | Slightly longer run time. |
| **Split into Alert vs Manual Review** | Clearer oversight & escalation. | More branches & nodes. |
| **Wait node before re-fetching** | Ensures ClickUp commits custom field updates. | Adds ~3 seconds per run. |
| **Loop-based update** | Guarantees updates even on variable data sizes. | More nodes in update pipeline. |
| **Strict fallback to manual review** | Zero silent failures. | Requires human intervention for malformed AI. |

---

# **4. Full Error Handling Strategy**

### *The system gracefully handles:*

- AI malformed JSON
- Missing fields
- Null blockers
- Bad percentages
- Undefined ClickUp fields
- Network delays
- ClickUp caching delays
- Duplicate alerts
- Duplicate comments
- Partial task updates

### Recovery Methods:

- AI retry mechanism
- Normalizer resets invalid values
- Manual review fallback
- Aggregated routing
- Type-checking in all Code nodes
- Timestamped audit log
- Fail-soft instead of fail-hard design

---

# **5. Anti-Duplicate & Anti-Spam Controls**

### **1. Email Duplication Fix**

Now sends exactly:

- 1 status alert email
- 1 manual review email
    
    (only if relevant)
    

### **2. Duplicate comment prevention**

A comment signature prevents repeated daily comments per task.

### **3. Prevent reprocessing**

Tasks processed today are tagged with a fresh timestamp.

### **4. Consolidation logic**

The `Aggregate Alerts` node collapses 30 items → 1.

This is what makes the system scale to 100+ tasks.

---

# **6. Output Examples (Clean & Structured)**

### **Status Alert Email Output**

Contains:

- Task Name
- Status Color
- Score
- Blockers
- AI Key Issues
- AI Recommendations
- Link to task

### **Manual Review Email Output**

Contains:

- A table-style summary of all tasks requiring review
- Reason for manual review (missing fields / invalid AI)
- Confirmation that a ClickUp “Manual Review Task” was created

---

# **7. Node Inventory (Simplified)**

Total nodes: 25**+**

Core Node Types Used:

- Trigger
- ClickUp (Get / Update / Create)
- Code (Normalize, Score, Validate, Aggregate)
- Gemini (AI)
- IF (Routing)
- Merge (Synchronize paths)
- Wait (Stabilization)
- Gmail (Notifications)
- Loop (Batch updates)

This is a *serious* pipeline, not a toy workflow.

---

# **8. Final Summary of system:**

- Fault tolerance
- AI augmentation
- Data validation
- Anti-spam routing
- Real-time task updates
- Clear escalation logic
- Full reporting
- End-to-end stability

---

**AI Prompt used HTTP body using Google Gemini 2.5 flash via API key** 

You are a project health analyst. Analyze this project and provide actionable insights in JSON format ONLY.

Project: {{projectName}}
Completion: {{completion}}%
Overdue Tasks: {{overdue}}
Blockers: {{blockers}}
My Assessment: Status is {{systemStatus}} with score {{score}}/100

RESPOND WITH ONLY THIS JSON (no markdown, no extra text):
{
"aiStatus": "RED" | "YELLOW" | "GREEN",
"isValid": true | false,
"keyIssues": "2-3 key problems (2 sentences max)",
"risks": "What happens if not fixed (2 sentences max)",
"recommendedActions": "Specific next steps (2-3 actions)"
}

Rules:

- aiStatus MUST match the system-calculated status ({{systemStatus}})
- isValid = true if you agree with the system, false if your assessment differs
- Be concise and actionable
- Return ONLY valid JSON, nothing else.

---
