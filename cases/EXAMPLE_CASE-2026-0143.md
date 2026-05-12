# CASE-2026-0143 — Sara M. → 1845 Westwood Dr

> Example case archive. Per-case files are git-ignored; this `EXAMPLE_` file ships with the repo so a newcomer sees the convention.

## Convention

One file per case. Latest envelope at the top. Prior envelopes below in reverse chronological order. The file IS the case state. The orchestrator scans this folder to detect existing `case_id`s.

When a case closes (sale closes, lead goes cold, deal terminates), append a `## Closed` block at the very bottom with the closing summary; the file stays in `cases/` as the audit trail.

---

## Latest

```json
{
  "schema_version": "1.0",
  "case_id": "CASE-2026-0143",
  "from": "04_transaction_coordinator",
  "to": "00_orchestrator",
  "back_to": null,
  "timestamp": "2026-06-25T16:14:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "inquiry_type": "state_change",
    "change_payload": {
      "event": "closed",
      "closing_date": "2026-06-25",
      "final_sale_price": 562000,
      "earnest_money_released_to": "seller via closing settlement",
      "outstanding_items": []
    },
    "current_risk_level": "🟢 Closed clean"
  },
  "required_fields_present": ["inquiry_type", "change_payload"],
  "confidence": "high",
  "next_action": "Move case to ## Closed block at bottom of file; archive trail.",
  "trail": ["00_orchestrator", "04_transaction_coordinator"]
}
```

## Prior envelopes (most-recent-first)

### 2026-06-15 — Back-handoff: 04_ → 03_ (appraisal delay)

```json
{
  "schema_version": "1.0",
  "case_id": "CASE-2026-0143",
  "from": "04_transaction_coordinator",
  "to": "03_client_communication",
  "back_to": "03_client_communication",
  "timestamp": "2026-06-15T09:14:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "draft_intent": "Heads-up to Sara: appraisal report delayed, name contingency date, name extension amendment as planned response, no panic.",
    "deadline_context": {
      "appraisal_report_expected_by": "2026-06-10",
      "appraisal_contingency_expires": "2026-06-18",
      "closing_date": "2026-06-25",
      "risk_level": "🟠 At-risk",
      "planned_response": "TREC extension amendment moving appraisal contingency out one week"
    }
  },
  "required_fields_present": ["draft_intent", "deadline_context"],
  "confidence": "high",
  "next_action": "03_ drafts; Diana sends; 04_ pre-stages amendment.",
  "trail": ["00_orchestrator", "04_transaction_coordinator"]
}
```

### 2026-06-02 — Open case

```json
{
  "schema_version": "1.0",
  "case_id": "CASE-2026-0143",
  "from": "00_orchestrator",
  "to": "04_transaction_coordinator",
  "back_to": null,
  "timestamp": "2026-06-02T11:30:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "inquiry_type": "open_case",
    "contract_document_reference": "executed_2026-06-01_1845_Westwood.pdf",
    "executed_dates": { "buyer": "2026-06-01", "seller": "2026-06-01" },
    "communication_date": "2026-06-02",
    "purchase_price": 562000,
    "earnest_money_amount": 5620,
    "option_period_days": 10,
    "option_fee_amount": 500,
    "closing_date": "2026-06-25",
    "financing_type": "conventional"
  },
  "required_fields_present": ["inquiry_type", "contract_document_reference", "executed_dates", "communication_date", "purchase_price", "earnest_money_amount", "option_period_days", "option_fee_amount", "closing_date", "financing_type"],
  "confidence": "high",
  "next_action": "Open transaction tracking; effective date = 2026-06-02 (one day after signing; PDF emailed morning of 2nd).",
  "trail": ["00_orchestrator"]
}
```

### 2026-05-11 — First reply drafted

(03_ produced draft per `03_client_communication/examples.md` Example 1. Sara replied same day, requested Saturday showing. Showing happened, offer drafted, executed June 1. Skipping the intermediate envelopes for brevity — in a real case file they'd all live here in order.)

### 2026-05-11 — Initial qualification

```json
{
  "schema_version": "1.0",
  "case_id": "CASE-2026-0143",
  "from": "01_lead_qualifier",
  "to": "02_property_research",
  "back_to": null,
  "timestamp": "2026-05-11T09:22:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "prospect_name": "Sara M.",
    "intent": "buy",
    "budget": "~550k",
    "timeline": "end of summer — Aug-Sep 2026",
    "location": ["78704"],
    "constraints": ["good schools", "2 kids (implies 3+ beds)"],
    "prior_relationship": "none — Zillow lead",
    "property_referenced": "1845 Westwood Dr, 78704",
    "gaps": ["pre-approval status", "financing type", "min beds/baths explicit", "current housing"]
  },
  "required_fields_present": ["prospect_name", "intent", "budget", "timeline", "location", "constraints", "prior_relationship", "gaps"],
  "confidence": "high",
  "next_action": "Pull comparables for 1845 Westwood and the 78704/78745 neighborhood; the brief feeds the first reply.",
  "trail": ["00_orchestrator", "01_lead_qualifier"]
}
```

### 2026-05-11 — Inbound landed in INTAKE.md

Zillow lead form, 09:14, no prior relationship. Raw text: *"Saw a house on Zillow I love at 1845 Westwood Dr 78704. Wondering if it's still available and what comparable homes have sold for nearby? Budget around 550k. Need to move by end of summer. Have two kids, so schools matter. Thanks! - Sara M."*

## Closed

Case closed 2026-06-25 at $562,000. 37 days from effective date to close. One back-handoff cycle (04_→03_→04_) for an appraisal delay; resolved with a one-week extension amendment. Earnest money $5,620 released to seller via closing settlement. No outstanding items. Sara moved in July 5.

Lessons logged for system reflection:
- Conventional-financing appraisal timing on credit-union loans runs slower than mortgage banks; consider noting lender-type → expected-appraisal-window in `04_/rules.md`.
- The pier-and-beam foundation note from `02_/examples.md` Example 1 surfaced an actual $4,200 issue during inspection that Sara had budgeted for. Domain detail in the research brief paid off.
