# Handoff — Lead Qualifier

> Schema reference: see root `/HANDOFF_SCHEMA.md`. This file only names my specifics.

## What I Receive

An envelope from `00_orchestrator/`.

### Required Payload Fields (the orchestrator owes me these)

- `raw_inbound` — the actual message text.
- `channel` — how it arrived (`zillow_lead_form`, `text`, `voicemail_transcript`, `email`, `referral`, `walk-in`, etc.).
- `received_at` — ISO-8601 timestamp.

### Optional Payload Fields

- `prospect_name` — if discoverable from the message.
- `property_referenced` — if the message names a specific address.
- `prior_relationship` — if the agent added context about a prior thread or case.

### If Required Fields Are Missing

If `raw_inbound` is missing or empty, I refuse the work and back-handoff to `00_orchestrator/` with `payload.routing_concern: "raw_inbound empty; nothing to qualify."` This is the only refusal scenario at this stage — `channel` and `received_at` I can infer (or mark unknown) and proceed.

## What I Produce

An envelope addressed to one of two specialists.

### Destinations

| Destination | When |
|---|---|
| `03_client_communication/` | Default. Next step is replying to the prospect. |
| `02_property_research/` | Only when the prospect named a specific property/neighborhood AND the agent's `next_action` requested research before reply. |

### Required Payload Fields on My Output

- `prospect_name` — if known; otherwise `"unknown"`.
- `intent` — `buy` / `sell` / `both` / `refi` / `other` / `unclear`.
- `budget` — string with prospect's words OR `"unknown"`.
- `timeline` — string with prospect's words OR `"unknown"`.
- `location` — array of ZIPs / neighborhoods / loose geographies OR empty array.
- `constraints` — array of strings (beds, baths, schools, financing, accessibility, pets, HOA) OR empty array.
- `prior_relationship` — string OR `"none"`.
- `gaps` — array of named gaps the next step needs to fill.

### Optional Payload Fields on My Output

- `intent_source` — explanation of how I read intent (especially when inferred).
- `inference_notes` — any inference the downstream specialist should understand.
- `property_referenced` — preserved from input.
- `location_inferred` — locations I inferred (e.g., similar school options) that the prospect did NOT name. Always flagged.

## Failure Modes

| Failure | What I do |
|---|---|
| `raw_inbound` empty | Back-handoff to `00_orchestrator/` with `routing_concern: "empty raw_inbound."` |
| Intent unclear (prospect could be vendor/agent/spam) | Back-handoff to `00_orchestrator/` with `routing_concern: "intent unclear — verify this is a real prospect."` |
| Prospect's first message is itself a request for a service-area decision (e.g., "Do you serve the Hill Country?") | I still capture what I can; I send to `03_client_communication/` with `confidence: med` and `next_action` noting the service-area question first. |

## Back-Handoffs To Me

`03_client_communication/` may back-handoff if a draft reveals the prospect's reply contains new qualifying data I should re-capture (e.g., "she said her budget is actually 700k, not 550k, after we talked"). I receive the back-handoff, update the qualified-lead record, and either:

- Re-send to `03_` for a revised draft, OR
- Re-send to `02_property_research/` if the new data materially changes the research scope.

In both cases the trail records that I was visited twice; downstream sees the qualification evolved.
