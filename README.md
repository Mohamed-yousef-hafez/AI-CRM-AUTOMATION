# AI-Powered CRM & Sales Reporting Automation

An end-to-end CRM automation built with **Make.com**. It receives leads, scores them with an LLM, flags leads that need follow-up, and produces KPI reports, with no manual work.

**Stack:** Make.com (scenarios, webhooks, data store) · Google Gemini (AI scoring) · JSON over HTTP

---

## What it does

| # | Scenario | Trigger | What happens |
|---|---|---|---|
| 1 | **AI Lead Intake & Scoring** | Webhook (website form or any HTTP call) | Lead data goes to an LLM, which returns a score (0-100), a category (Hot / Warm / Cold), a short reason and a recommended next action. The lead is saved in the CRM and the result is returned in real time. **Hot leads also trigger an instant Gmail alert** to the sales owner. |
| 2 | **Daily Follow-up Check** | Schedule, every day 09:00 | Finds leads with status `New` and no contact for 5+ days, and changes their status to `Follow-up Needed`. |
| 3 | **KPI Report** | Webhook (on demand) | Reads the CRM and returns total leads, Hot/Warm/Cold counts, follow-ups needed, pipeline value and average score as JSON. |

## Screenshots

**The three scenarios in Make** (Workflow 2 is switched off because the free plan allows only two active scenarios):

![Scenarios](screenshots/1-scenarios.png)

## Architecture

```
Website form / HTTP call
        |
   [Webhook]  --->  [Gemini AI: score + category + reason + next action]
                              |---- (Hot only) ---> [Gmail alert]
                              |
                     [CRM Data Store]  <--- [Daily follow-up check]
                              |
                     [KPI report endpoint]
```

## CRM data model

`name, company, email, job_title, source, budget, message, score, category, ai_reason, next_action, status, value, owner, created_at, last_contact`

- The lead's email is the record key, so a repeated submission updates the same lead instead of creating a duplicate.
- Hot leads are assigned to `Senior Sales`, all others to `Sales Rep`.

## Example

Request:

```
GET <your-webhook-url>?name=Mona&company=TechCo&email=mona@techco.com
    &job_title=Operations Manager&budget=30000&source=LinkedIn
    &message=We want to automate our sales reporting for a team of 20 people
```

Response:

```json
{
  "result": "saved",
  "score": 88,
  "category": "Hot",
  "reason": "The lead has a clear need, a strong budget, and an appropriate contact.",
  "next_action": "Schedule a discovery call to understand their current reporting challenges."
}
```

KPI report response:

```json
{"total_leads":3,"hot":1,"warm":1,"cold":1,"followups_needed":1,"pipeline_value":80500,"avg_score":63}
```

## AI scoring prompt (simplified)

> You are a B2B sales expert. Evaluate this lead from 0 to 100 based on company size, job title seniority, budget and clarity of the request. Reply with one line only in the format `score|category|reason|next_action`. Hot is 70+, Warm is 40-69, Cold is below 40.

The reply is split on `|` and each part is stored in its own CRM field.

## How to deploy it (for a team or client)

1. **Create a Make account** (free plan is enough for a demo) and a Data Store using the CRM fields above.
2. **Import the blueprints** from the `blueprints/` folder: *Create scenario → ... menu → Import Blueprint*.
3. **Fill the placeholders** in each imported scenario: `<YOUR_WEBHOOK_ID>` (create a Custom Webhook), `<YOUR_DATASTORE_ID>` (pick your Data Store), `<YOUR_GMAIL_CONNECTION_ID>` and `<YOUR_EMAIL>` (connect Gmail and set the sales inbox).
4. **Connect the lead source:** point the website form (or Google Forms / any tool that can call a URL) to the webhook URL.
5. **Test** with a sample lead, check the CRM record and the Hot-lead email, then switch the scenarios on.

Blueprints are sanitized exports. Module settings may need a quick re-selection after import.

## Positioning

This project is an **automation layer**, not a replacement for a full CRM such as Odoo, HubSpot or Salesforce. In a real company the Data Store would be swapped for the company's CRM, keeping the AI scoring, alerts, follow-up logic and reporting on top of it.

## Limitations and next steps

- This is a working prototype / portfolio project, not a production system: the webhook has no authentication, the Data Store is small, and there is no user interface.
- On Make's free plan only two scenarios can be active at once, so scenario 2 and 3 are switched on as needed.
- Follow-up currently changes the lead status only. Next step: send a Gmail draft to the sales owner for review (human-in-the-loop).
- Next step: a scheduled weekly report sent by email, and a web form in front of the webhook.
- AI scores are a first-pass signal for a salesperson, not a final decision.

## What I learned

- Designing webhook-based automations and handling real data types between modules.
- Prompting an LLM to return a strict, parseable format.
- Debugging scenarios with execution logs and replays (for example, fixing an unsupported function in a scoring formula).
