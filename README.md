# AI Customer Support Automation Platform

An end-to-end customer support automation system built entirely in **n8n** — combining webhook triggers, scheduled crons, AI-powered decision making, conditional branching, and a human-in-the-loop resolution step across **5 independent workflows** and **35+ nodes**.

> n8n Capstone Project | Built by [Kapil Yadav]

---

## 📌 Overview

Growing businesses that handle customer support manually struggle to scale — tickets go unacknowledged, get misrouted, urgent issues slip through, and there's no consistent feedback loop or reporting. This project automates the **entire support ticket lifecycle**, from intake to resolution to weekly analytics, using AI for classification and decision-making while keeping a human agent in control of marking issues resolved.

**Read the full [Project Report](./AI_Customer_Support_Automation_Project_Report.docx)** for detailed problem analysis, architecture, and implementation notes.

---

## 🏗️ Architecture

![System Architecture](./architecture_diagram.png)

The platform uses **Google Sheets** as a shared data layer (two tabs: `Tickets` and `Feedback`) so that each workflow can be built, tested, and modified independently. Workflows communicate indirectly — one workflow's write triggers the next workflow's read.

### Event Flow

| Step | Event | Workflow Triggered |
|---|---|---|
| 1 | Customer submits a request (webhook) | **Workflow 1** — Ticket Collection & Registration |
| 2 | New row added to `Tickets` sheet | **Workflow 2** — AI Classification & Prioritization |
| 3 | Category/Priority updated | **Workflow 3** — Agent Assignment & Escalation |
| 4 | Agent sets Status = "Resolved" | **Workflow 4** — Customer Communication & Resolution |
| 5 | Feedback submitted (webhook) + weekly Cron | **Workflow 5** — Feedback & Analytics |

---

## ⚙️ Workflows

| # | Workflow | Trigger | Nodes | File |
|---|---|---|---|---|
| 1 | Ticket Collection & Registration | Webhook (POST) | 5 | [`Workflow1_Ticket_Collection.json`](./Workflow1_Ticket_Collection.json) |
| 2 | AI Ticket Classification & Prioritization | Google Sheets Trigger | 5 | [`Workflow2_AI_Classification.json`](./Workflow2_AI_Classification.json) |
| 3 | Agent Assignment & Escalation | Google Sheets Trigger | 9 | [`Workflow3_Agent_Assignment.json`](./Workflow3_Agent_Assignment.json) |
| 4 | Customer Communication & Resolution | Google Sheets Trigger | 6 | [`Workflow4_Customer_Resolution.json`](./Workflow4_Customer_Resolution.json) |
| 5 | Feedback & Analytics | Webhook + Weekly Cron | 10 | [`Workflow5_Feedback_Analytics.json`](./Workflow5_Feedback_Analytics.json) |

**Total: 35 nodes across 5 workflows**

### Workflow 1 — Ticket Collection & Registration
Receives a customer support request via webhook, generates a unique Ticket ID, stores it in the `Tickets` sheet, and sends a confirmation email.

### Workflow 2 — AI Ticket Classification & Prioritization
Triggered when a new ticket row appears. Sends the ticket's subject and description to an LLM (Groq — Llama 3.3 70B), which returns structured JSON with `Category`, `Priority`, `Sentiment`, and `Reason`. The result updates the same row.

### Workflow 3 — Agent Assignment & Escalation
Triggered when Category/Priority are set. A Switch node routes the ticket to the Technical, Billing, Sales, or General team based on Category. If Priority is `High`, an additional escalation email is sent to the manager.

### Workflow 4 — Customer Communication & Resolution
Triggered when a support agent manually sets Status = `Resolved`. Sends a resolution email to the customer, waits, then sends a feedback request email. Marks the row `Resolved - Notified` to prevent duplicate sends.

### Workflow 5 — Feedback & Analytics
Two independent triggers in one workflow:
- **Webhook branch**: logs incoming customer feedback (rating + comments) to the `Feedback` sheet.
- **Weekly Cron branch** (every Monday, 9 AM): reads both sheets, aggregates statistics (ticket counts, resolution rate, average rating), asks the AI model to write a natural-language summary, and emails it to the manager.

---

## 🧠 Advanced Features

- **AI-powered decision making** — Groq Llama 3.3 classifies tickets and writes the weekly report
- **Human-in-the-loop approval** — Workflow 4 only fires after an agent manually marks a ticket resolved
- **Conditional branching** — IF nodes and a 4-way Switch node throughout
- **Webhook-triggered workflows** — ticket intake and feedback intake
- **Scheduled workflow (Cron)** — weekly analytics report every Monday 9 AM
- **Error handling & fallback** — AI JSON parsing wrapped in try/catch with safe defaults
- **Logging & audit trail** — full ticket lifecycle persisted in Google Sheets
- **Idempotency guards** — "already classified / already assigned" checks prevent duplicate processing

---

## 🛠️ Tech Stack

- **Automation platform:** [n8n](https://n8n.io) (Cloud)
- **Data store:** Google Sheets
- **Email:** Gmail (OAuth2)
- **AI model:** Groq API — Llama 3.3 70B Versatile (OpenAI-compatible endpoint)

---

## 🚀 Setup Instructions

1. **Import the workflows**
   - In n8n, go to **Workflows → Import from File** and import each of the 5 JSON files in this repo.

2. **Create a Google Sheet** named `AI Customer Support Master Sheet` with two tabs:
   - `Tickets` — columns: `Ticket ID, Customer, Email, Subject, Description, Category, Priority, Assigned Team, Status, Date`
   - `Feedback` — columns: `Ticket ID, Rating, Feedback, Date`

3. **Connect credentials** in each workflow:
   - **Google Sheets OAuth2** — for all Google Sheets nodes
   - **Gmail OAuth2** — for all email nodes
   - **Groq API key** — added as an "OpenAI API" credential type in n8n, with **Base URL** set to `https://api.groq.com/openai/v1` ([get a free key here](https://console.groq.com))

4. **Update placeholders** in each workflow:
   - Replace the Google Sheet ID in every Sheets node with your own
   - Replace placeholder team/manager email addresses in the Gmail nodes

5. **Activate** each workflow (top-right **Publish/Active** toggle) so triggers run automatically.

6. **Test end-to-end** by sending a POST request to Workflow 1's webhook URL:
   ```json
   {
     "name": "Test Customer",
     "email": "you@example.com",
     "subject": "Login issue",
     "description": "Unable to log in since yesterday",
     "attachment": ""
   }
   ```

---

## 📂 Repository Contents

```
├── Workflow1_Ticket_Collection.json
├── Workflow2_AI_Classification.json
├── Workflow3_Agent_Assignment.json
├── Workflow4_Customer_Resolution.json
├── Workflow5_Feedback_Analytics.json
├── architecture_diagram.png
├── AI_Customer_Support_Automation_Project_Report.docx
├── AI_Customer_Support_Automation_Presentation.pptx
└── README.md
```

---

## 🎥 Demo Video

📺 [Watch the demo](#) — *(add your video link here after recording)*

---

## 📄 Documentation

- **[Project Report](./AI_Customer_Support_Automation_Project_Report.docx)** — Problem analysis, architecture, implementation details, testing summary
- **[Presentation Deck](./AI_Customer_Support_Automation_Presentation.pptx)** — 12-slide summary

---

## 🔮 Future Enhancements

- Replace polling-based Google Sheets triggers with a real database + native change webhooks
- Add Slack / Microsoft Teams notifications alongside email
- Auto-parse customer feedback email replies using AI instead of a structured webhook
- Add a centralized Error Trigger workflow for failure alerts
- Build a live analytics dashboard (e.g., Looker Studio) on top of the existing data

---

## 👤 Author

**[Kapil Yadav]**

