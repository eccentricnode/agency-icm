# Handoff — Property Research

> Schema reference: see root `/HANDOFF_SCHEMA.md`. This file only names my specifics.

## What I Receive

An envelope from `00_orchestrator/` (for standalone research requests) or `01_lead_qualifier/` (for qualified-lead-driven research).

### Required Payload Fields

- `query_subject` — a specific address, a neighborhood, a ZIP, or a market segment label.
- `query_type` — `comparables` / `neighborhood` / `market_read` / `valuation_sketch` / `pre_listing_diligence`.

### Optional Payload Fields

- `prior_case_id` — if continuing research on an existing case.
- `comparables_window` — override default 6 months (e.g., `12_months` or `same_quarter_prior_year`).
- `school_focus` — if the prospect named school constraints.
- `agent_on_deal` — drives whose lived-experience anecdotes I'm allowed to attribute.

### If Required Fields Are Missing

If `query_subject` or `query_type` is missing, I back-handoff to the sender with `routing_concern: "research request needs a subject and a query_type; current request is too vague to scope."`

## What I Produce

An envelope addressed to one of:

| Destination | When |
|---|---|
| `03_client_communication/` | Default. Brief feeds into client-facing message draft. |
| `00_orchestrator/` (back-handoff) | If the case is under contract — the request is misrouted to me. |
| `01_lead_qualifier/` (back-handoff) | Rare. If the research surfaces that the qualification was wrong (e.g., budget shifts after a market read). |

### Required Payload Fields on My Output

- `brief_subject` — the subject of the brief.
- `agent_pull_quotes` — array of 2-4 one-liners the agent can quote to the client. These are the value-extraction.
- `sources_named` — array of source labels I used. `03_` refuses to draft from a brief with empty `sources_named`.

### Optional Payload Fields on My Output

- `comparable_count` — number of closed comparables.
- `comparable_range`, `comparable_median`.
- `neighborhood`, `school_zone`.
- `implied_net` (sell-side).
- `trec_form_implied` — informational; helps `04_` later.

## Failure Modes

| Failure | What I do |
|---|---|
| `query_subject` is too vague (e.g., "Austin housing market") | Back-handoff to sender with `routing_concern: "subject too broad; need a ZIP, neighborhood, or address."` |
| Comparable count < 3 | Send anyway with `confidence: low` and `next_action` instructing `03_` not to quote my numbers without re-checking. |
| Sources unavailable (MLS access down, etc.) | Send with `confidence: low`, `sources_named: ["secondary only — Zillow public, Realtor.com public"]`, flag in `next_action`. |
| Case is under contract | Back-handoff to `00_orchestrator/` with `routing_concern: "case is under contract; this is 04_'s domain."` |

## Back-Handoffs To Me

`03_client_communication/` may back-handoff if the agent's preferred reply requires a comparable or stat I didn't pull. I receive the back-handoff, run the targeted query, and re-send to `03_` with the added data.

`01_lead_qualifier/` may back-handoff after `03_`'s draft prompts a prospect reply that changes the qualification, which in turn changes the research scope. Trail records the loop honestly.
