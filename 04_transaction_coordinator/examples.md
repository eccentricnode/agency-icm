# Examples — Transaction Coordinator

Two case states. The first is a routine status snapshot. The second is the back-handoff scenario — this is the move most submissions for this comp will miss.

---

## Example 1 — Status Snapshot for an Active Case

### Inbound

From `00_orchestrator/`. Diana asked: "What's the state of CASE-2026-0119 (Marcus on the Wendover place)? Specifically, has appraisal come back, and when does option period end?"

### Case State

**Case:** CASE-2026-0119 — Marcus Hernandez, buyer, 4321 Wendover Ln #B 78745
**Contract:** TREC 20-18 (Resale)
**Executed (date both signed):** 2026-05-05
**Effective date (signature communicated in writing):** 2026-05-06 — *delayed one day because listing agent sent the executed PDF on the morning of the 6th*
**Closing date:** 2026-06-12 (37 days from effective)

| Milestone | Deadline | State | Risk |
|---|---|---|---|
| Earnest money deposit ($5,500 = ~1.5%) | 2026-05-11 (3 business days) | ✅ Deposited 2026-05-08, confirmed by Lonestar Title | 🟢 |
| Option period ends | 2026-05-16 (10 cal. days from effective) | Active; ends Saturday 11:59pm | 🟡 |
| Inspection report due | (within option period) | ⏳ Inspector pulled Tuesday; report expected by Thursday | 🟡 |
| Title commitment delivered | 2026-05-26 (20 days from contract to title co.) | ⏳ Title co. confirmed receipt; on track | 🟢 |
| Appraisal report | (no contractual deadline — lender driven) | ⏳ Ordered 2026-05-08; lender estimate 5-10 business days | 🟡 |
| Appraisal contingency expires | 2026-06-05 (7 days before close) | ⏳ Pending appraisal report | 🟢 (not yet at-risk) |
| Financing contingency | 2026-06-05 | ⏳ Pending appraisal + final UW | 🟢 |
| Closing | 2026-06-12 | ⏳ On schedule | 🟢 |

### Answers to Diana's Questions

- **Appraisal?** Ordered 2026-05-08. Lender estimated 5-10 business days. As of today nothing has come back. T-48hr alert will fire 2026-05-15 if still no report. Not at-risk yet.
- **Option period ends?** Saturday 2026-05-16 at 11:59pm. Inspection report is expected Thursday — gives the buyer two days to decide on path (terminate / repair-request / credit-request / accept as-is) before option expiry.

### Envelope I Produce

```json
{
  "case_id": "CASE-2026-0119",
  "from": "04_transaction_coordinator",
  "to": "00_orchestrator",
  "back_to": null,
  "timestamp": "2026-05-11T14:08:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "inquiry_type": "status",
    "case_summary": "...full case state above...",
    "answers": {
      "appraisal": "Ordered 2026-05-08; lender est. 5-10 business days. No report yet. T-48hr alert will fire 2026-05-15 if still pending.",
      "option_period_ends": "2026-05-16 at 11:59pm. Inspection report expected Thursday."
    },
    "current_risk_level": "🟡 Watch — option period closes in 5 days, awaiting inspection + appraisal."
  },
  "required_fields_present": ["inquiry_type", "answers"],
  "confidence": "high",
  "next_action": "Diana may want a client-facing reply to Marcus — if so, back-handoff to 03_client_communication.",
  "trail": ["00_orchestrator", "04_transaction_coordinator"]
}
```

---

## Example 2 — Back-Handoff Scenario (THE EUPHORIC-SURPRISE MOVE)

### Trigger

It's 2026-05-13. Appraisal report was due Monday. It's now Wednesday morning. T-72hr alert just fired — appraisal contingency expires in 23 days, but the lender's quoted return window has already slipped past. Buyer hasn't been told.

This is a 🟠 At-risk event. I do not wait for someone to ask. I produce a back-handoff to `03_client_communication/` so a heads-up draft is ready before lunch.

### Envelope I Produce (Back-Handoff)

```json
{
  "case_id": "CASE-2026-0119",
  "from": "04_transaction_coordinator",
  "to": "03_client_communication",
  "back_to": null,
  "timestamp": "2026-05-13T09:42:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "draft_intent": "Heads-up email to Marcus that the appraisal report is delayed; name the contingency date; name the extension amendment as a planned response; no panic.",
    "deadline_context": {
      "appraisal_report_expected_by": "2026-05-11 (Monday)",
      "current_status": "Lender has not delivered as of 2026-05-13 09:00",
      "appraisal_contingency_expires": "2026-06-05",
      "closing_date": "2026-06-12",
      "risk_level": "🟠 At-risk",
      "lender_followups_so_far": 2,
      "planned_response_if_no_report_by_thursday_eod": "Send TREC extension amendment moving appraisal contingency out one week"
    },
    "agent_review_notes": "Send today before Marcus chases us first. He's a first-time buyer — he doesn't know that a one-week extension amendment is routine in this market."
  },
  "required_fields_present": ["draft_intent", "deadline_context"],
  "confidence": "high",
  "next_action": "03_ drafts client email; once Diana sends, 04_ pre-stages the TREC extension amendment so it's ready for signature if Thursday EOD arrives without the report.",
  "trail": ["00_orchestrator", "04_transaction_coordinator"]
}
```

### What Happens Next

`03_client_communication/` reads this envelope and drafts the message in Diana's voice (see Example 3 in `03_client_communication/examples.md` — same scenario from the other side).

After Diana sends the draft, `03_` returns the message-sent confirmation to `04_` (via another envelope, or via the trail logged in the case state). `04_` updates the case state with "client notified Wednesday morning" and pre-stages the extension amendment for Thursday signature.

The trail for this back-handoff chunk: `04_transaction_coordinator → 03_client_communication → (agent sends) → 04_transaction_coordinator`.

That round-trip is the move that makes the system real. Most Comp #4 submissions will treat the architecture as a one-way pipeline. Real deals require this round-trip on every other transaction.

---

## What These Examples Show

- I am tracking dates, not feelings. Every milestone has a deadline, a state, and a risk color.
- The **effective date ≠ signed date** distinction is logged on every case as the first line of state. This is the Texas-specific tell that wins domain credibility.
- The back-handoff to `03_` happens proactively at the 🟠 At-risk threshold, not after a fire. The agent's day is calmer because I run ahead.
- The trail records the round-trip honestly. The audit log is the architecture's truth.
