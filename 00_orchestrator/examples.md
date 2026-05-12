# Examples — Orchestrator

Two worked routing examples. Each shows the raw inbound, my reasoning, and the envelope I produce in both the human-writable form (markdown table) and the machine-parseable form (JSON-in-fence). Downstream specialists can read either.

---

## Example 1 — New Zillow Lead

### Raw Inbound (from `INTAKE.md`)

> Saw a house on Zillow I love at 1845 Westwood Dr 78704. Wondering if it's still available and what comparable homes have sold for nearby? Budget around 550k. Need to move by end of summer. Have two kids, so schools matter. Thanks! - Sara M.
>
> *Channel*: Zillow lead form, received 2026-05-11 09:14
> *Agent context*: No prior relationship.

### My Reasoning

- No existing `case_id` mentions Sara M. or 1845 Westwood. → new case.
- New prospect message → routing rule 4 → `01_lead_qualifier/`.
- Required fields for `01_`: `raw_inbound`, `channel`, `received_at`. All present.
- I do NOT pre-qualify her even though her intent (buy), budget (550k), timeline (summer), and one constraint (schools) are obvious from the message. That's `01_`'s job.

### Envelope I Produce — Human-Writable Form

| Field | Value |
|---|---|
| case_id | CASE-2026-0143 |
| from | 00_orchestrator |
| to | 01_lead_qualifier |
| back_to | null |
| timestamp | 2026-05-11T09:18:00-05:00 |
| agent_on_deal | Diana |
| confidence | high |
| next_action | Qualify Sara M. on intent (buy implied), budget (~550k stated), timeline (end of summer), location (78704), school-importance constraint. |
| trail | 00_orchestrator |

**Payload**

| Payload field | Value |
|---|---|
| raw_inbound | Saw a house on Zillow I love at 1845 Westwood Dr 78704... [full text preserved] |
| channel | zillow_lead_form |
| received_at | 2026-05-11T09:14:00-05:00 |
| prospect_name | Sara M. |
| property_referenced | 1845 Westwood Dr, 78704 |

### Envelope I Produce — JSON Form

```json
{
  "schema_version": "1.0",
  "case_id": "CASE-2026-0143",
  "from": "00_orchestrator",
  "to": "01_lead_qualifier",
  "back_to": null,
  "timestamp": "2026-05-11T09:18:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "raw_inbound": "Saw a house on Zillow I love at 1845 Westwood Dr 78704...",
    "channel": "zillow_lead_form",
    "received_at": "2026-05-11T09:14:00-05:00",
    "prospect_name": "Sara M.",
    "property_referenced": "1845 Westwood Dr, 78704"
  },
  "required_fields_present": ["raw_inbound", "channel", "received_at"],
  "confidence": "high",
  "next_action": "Qualify Sara M. on intent (buy implied), budget (~550k stated), timeline (end of summer), location (78704), school-importance constraint.",
  "trail": ["00_orchestrator"]
}
```

---

## Example 2 — Status Check on a Signed Deal

### Raw Inbound (from `INTAKE.md`)

> Marcus (the buyer on the Wendover place) just texted asking whether the appraisal came back yet and what the option period ends. Can you check?
>
> *Channel*: internal Slack from Diana, 2026-05-11 14:02
> *Agent context*: Case CASE-2026-0119, under contract, option period still open as of yesterday.

### My Reasoning

- Existing `case_id` (CASE-2026-0119) referenced. → not a new case.
- Status check on a signed deal → routing rule 1 → `04_transaction_coordinator/`.
- Required fields for `04_`: `case_id`, `inquiry_type`. Both present.
- This is NOT a request to draft a message — Diana asked me to "check," not "draft a reply." If `04_` returns with status, the orchestrator may then route to `03_` to draft a reply. That would be a back-handoff scenario; see `04_transaction_coordinator/handoff.md`.

### Envelope I Produce — JSON Form

```json
{
  "schema_version": "1.0",
  "case_id": "CASE-2026-0119",
  "from": "00_orchestrator",
  "to": "04_transaction_coordinator",
  "back_to": null,
  "timestamp": "2026-05-11T14:04:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "inquiry_type": "status",
    "questions": [
      "Has the appraisal come back yet?",
      "When does the option period end?"
    ],
    "asker": "Marcus (buyer)"
  },
  "required_fields_present": ["inquiry_type"],
  "confidence": "high",
  "next_action": "Pull current status on appraisal and option period for CASE-2026-0119. If a client-facing reply is needed, back-handoff to 03_client_communication after status is confirmed.",
  "trail": ["00_orchestrator"]
}
```

---

## What Both Examples Show

- I keep `next_action` short but specific — the receiver knows exactly what I think comes next.
- I never put work in the payload that the downstream specialist owns.
- I append `00_orchestrator` to the trail before sending.
- I include both forms in this file because the human types in markdown and the LLM reads either form fluently.
