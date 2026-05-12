# Handoff — Transaction Coordinator

> Schema reference: see root `/HANDOFF_SCHEMA.md`. This file describes my specifics — input shape, output shape, **and the back-handoff protocol that makes the system bi-directional.**

## What I Receive

An envelope from `00_orchestrator/` on three kinds of triggers:

1. **Case-opening** — a contract has been executed; the case moves from research/communication phase into transaction phase. Required payload fields: `case_id`, `inquiry_type: "open_case"`, `contract_document_reference`, `executed_dates` (both parties' physical signature dates), `communication_date` (when final signature was communicated in writing — the Texas-specific gotcha).
2. **Status check** — the agent or a client wants a snapshot. Required: `case_id`, `inquiry_type: "status"`, optionally `questions` (specific things to answer).
3. **State change** — something happened (report received, signature signed, deadline shifted, amendment signed). Required: `case_id`, `inquiry_type: "state_change"`, `change_payload` (what happened).

### Required Payload Fields by Inquiry Type

| Inquiry | Required fields |
|---|---|
| `open_case` | `case_id`, `contract_document_reference`, `executed_dates`, `communication_date`, `purchase_price`, `earnest_money_amount`, `option_period_days`, `option_fee_amount`, `closing_date`, `financing_type` (`conventional` / `FHA` / `VA` / `cash` / `owner_financed` / `assumption` / other). For `owner_financed`, additionally requires `owner_finance_terms: { down_payment_pct, note_amount, interest_rate, amortization_years, balloon_date \| null, prepayment_penalty }`. |
| `status` | `case_id`, `inquiry_type` |
| `state_change` | `case_id`, `inquiry_type`, `change_payload` |

### If Required Fields Are Missing

If `open_case` is missing `communication_date`, I back-handoff to `00_orchestrator/` with `routing_concern: "communication_date not provided; without it, contingency clocks cannot start. Confirm the date the final signature was communicated in writing."`

## What I Produce

Two distinct output shapes:

### Shape A — Status / State Envelope (forward)

Addressed to `00_orchestrator/`. Payload carries the case state, answers to any specific questions, and the current risk level.

### Shape B — Back-Handoff to `03_client_communication/`

When a deadline is at-risk or a client-facing message is needed. Payload carries `draft_intent`, `deadline_context` (full set of dates and risks), and `agent_review_notes`.

**This back-handoff is first-class.** It is not an exception, not an error, not a workaround. Real transactions require it 1-3 times per deal on average. Modeling the system as one-way breaks the day a real deal moves.

### Required Payload Fields on Output

| Output type | Required |
|---|---|
| Status (Shape A) | `case_summary`, `current_risk_level` |
| State change ack | `case_id`, `state_after_change` |
| Back-handoff to `03_` (Shape B) | `draft_intent`, `deadline_context` |

## Failure Modes

| Failure | What I do |
|---|---|
| `communication_date` not provided on `open_case` | Back-handoff to `00_orchestrator/` requesting confirmation |
| Earnest money not deposited within 3 business days of effective date | Back-handoff to `03_client_communication/` for a notice to the other side's agent; risk-level 🟠 |
| Option period passes without buyer notice (termination, repair-request, or proceed) | Back-handoff to `03_` for client-facing confirmation; case auto-moves to "post-option" state |
| Title commitment not delivered by deadline | Back-handoff to `03_` for follow-up; risk-level 🟠 |
| Appraisal report delayed past lender estimate | Back-handoff to `03_` at T-72hr; pre-stage TREC extension amendment |
| Closing date moves | `state_change` envelope to `00_orchestrator/`; back-handoff to `03_` for all-parties notification |
| Wire instructions or signing details unresolved at T-72hr | Back-handoff to `03_` for signing-coordination message |

## Back-Handoff Triggers Cheat Sheet

```
T-72hr → log + watch
T-48hr → back-handoff to 03_ (draft prepared)
T-24hr → back-handoff to 03_ (urgent) + agent phone call
Deadline missed → back-handoff to 03_ (damage control) + agent phone call (cannot be only text)
```

## A Note on the Trail

A typical transaction trail for me looks like:

```
00_orchestrator → 04_transaction_coordinator
   ↓
   ... days pass, milestones logged ...
   ↓
04_transaction_coordinator → 03_client_communication   (back-handoff: appraisal delay)
03_client_communication → (agent sends draft)
   ↓ (state change envelope)
00_orchestrator → 04_transaction_coordinator
   ↓
   ... closing approaches ...
   ↓
04_transaction_coordinator → 03_client_communication   (back-handoff: closing-day coordination)
```

That round-trip is the architecture working as intended. The trail is the audit log. The audit log is the truth.

## Why Back-Handoffs Matter

A multi-agent system that doesn't model back-handoffs is a pipeline pretending to be a team. Real transactions loop — the coordinator catches a deadline slip and the communication specialist sends the heads-up; later the coordinator catches a signing-day need and the communication specialist coordinates the time. The shape of the trail will show those loops if the architecture allows them.
