# Handoff — Orchestrator

> Schema reference: see root `/HANDOFF_SCHEMA.md`. This file describes only what's specific to me — what I receive, what I produce, and how I fail safely.

## What I Receive

I am the entry point. I do not receive an envelope from another specialist (except on back-handoffs, see below). My input is:

| Source | Shape |
|---|---|
| `/INTAKE.md` | A free-form paste of a raw inbound request, the channel it arrived on, the timestamp, and a one-line context note from the human agent (optional). |
| Existing case files | If a `case_id` already exists in the system, I read its prior `trail` and any prior envelopes. |

### Required Fields on Input (the human's responsibility)

- A `### Raw Inbound` section in `INTAKE.md` with the actual message.
- A `Channel` line.
- A `Received` date/time.

### Optional Fields on Input

- `Agent context` — one line of human-added context the message itself doesn't carry.
- `Received by` — which agent on the team caught it.

## What I Produce

A single envelope, conforming to `/HANDOFF_SCHEMA.md`, addressed to one of the four downstream specialists.

### Required Payload Fields by Destination

I do NOT add fields the destination doesn't list. I do NOT skip fields the destination lists. If `INTAKE.md` doesn't contain a destination's required field, I either route to a different specialist or send with `confidence: low` and name the gap.

| Destination | Required payload fields | Optional payload fields |
|---|---|---|
| `01_lead_qualifier/` | `raw_inbound`, `channel`, `received_at` | `prospect_name`, `property_referenced`, `prior_relationship` |
| `02_property_research/` | `query_subject`, `query_type` | `prior_case_id`, `comparables_window`, `school_focus` |
| `03_client_communication/` | `case_id`, `agent_on_deal`, `draft_intent` | `prior_thread_summary`, `tone_override` |
| `04_transaction_coordinator/` | `case_id`, `inquiry_type` | `questions`, `asker`, `deadline_focus` |

## Failure Modes

| Failure | What I do |
|---|---|
| `INTAKE.md` is missing a `### Raw Inbound` section | I do not produce an envelope. I write a note back into `INTAKE.md`'s `## Orchestrator Notes` section asking the human to paste the actual inbound. |
| Two specialists could plausibly handle this | I route to `01_lead_qualifier/` with `confidence: low`. The lead qualifier is the closest thing to a human triage step the system has. |
| A required field for the chosen destination is missing from `INTAKE.md` | I either re-route to a specialist that captures that field first, or I send with `confidence: low` and name the gap in `next_action`. |
| A downstream specialist back-handoffs me with "wrong routing" | I read their feedback in `next_action`, re-route from scratch, and append both `00_orchestrator` entries to the trail (the second one for the re-route). |

## Back-Handoffs To Me

The other four specialists can back-handoff to me only for routing disputes. They do NOT back-handoff to me to ask the human a question — that's their own job. If a back-handoff arrives, I treat the envelope's `payload.routing_concern` as authoritative and re-route.

Example: `02_property_research` receives a request to research "comparables for the Marcus Wendover deal" and discovers the case has no prior research and `case_id` looks stale. It back-handoffs to `00_orchestrator` with `payload.routing_concern: "stale case_id; verify this is the active deal."` I either confirm or open a new case.

## Note on the Trail

I append `00_orchestrator` to the trail BEFORE I send. Every receiver sees my role in the path so far. On re-routes I append twice (once per routing decision) — the audit log should show I had to think twice.
