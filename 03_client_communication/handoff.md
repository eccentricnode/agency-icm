# Handoff — Client Communication

> Schema reference: see root `/HANDOFF_SCHEMA.md`. This file only names my specifics.

## What I Receive

An envelope from one of:

- `01_lead_qualifier/` — for first-reply drafts.
- `02_property_research/` — when research feeds a client message.
- `04_transaction_coordinator/` — for deadline-slip, status-update, or signing-coordination messages. **This is a back-handoff path; trail records it.**

### Required Payload Fields

- `case_id` — the case this message belongs to.
- `agent_on_deal` — the agent whose voice the draft must match.
- `draft_intent` — one sentence on what this message is supposed to accomplish (first reply, confirm showing, send competing-offer strategy, etc.).

### Optional Payload Fields

- `agent_pull_quotes` (from `02_`) — concrete data I should incorporate.
- `deadline_context` (from `04_`) — dates and risks I should reference.
- `prior_thread_summary` — what's been said before, so I don't repeat.
- `tone_override` — rare; if the agent wants me to bypass the voice file's defaults (e.g., the deal is shaky, agent wants more formal).
- `channel` — `email` / `text` / `voicemail_talking_points`.

### If Required Fields Are Missing

- Missing `agent_on_deal` → back-handoff to sender; I cannot draft in no-voice.
- Missing `case_id` → back-handoff; case context is required.
- Missing `draft_intent` → I produce a generic outreach and flag `confidence: low`; agent decides.
- Missing voice file for the named agent → graceful degrade. Draft in house voice (`voice/EXAMPLE_voice_diana.md` baseline), `confidence: low`, `do_not_send_yet: ["voice not captured for <agent>"]`, plus a secondary envelope to `00_orchestrator/` tagged `payload.onboarding_gap` so the missing voice gets captured async. Never block on missing voice.

## What I Produce

An envelope addressed to one of:

| Destination | When |
|---|---|
| `00_orchestrator/` | Default. The orchestrator returns the draft to the agent for review-and-send. |
| `04_transaction_coordinator/` | When my draft is a transaction-stage message and `04_` needs to know I drafted it (post-execution paperwork pre-staging). |
| `01_lead_qualifier/` (back-handoff) | When the client's reply (received later, fed back through `INTAKE.md`) contains new qualifying data. |
| `02_property_research/` (back-handoff) | When my draft needs a comparable or stat I don't have. |

### Required Payload Fields on My Output

- `channel` — `email` / `text` / `voicemail_talking_points`.
- `draft_primary` — the actual message text the agent will send.

### Optional Payload Fields on My Output

- `alternates` — array of alternate drafts when situation is sensitive (default empty).
- `do_not_send_yet` — array of items the agent must verify before sending.
- `agent_review_notes` — meta-commentary for the agent (why I made the choices I made).

## Failure Modes

| Failure | What I do |
|---|---|
| Voice file missing | Back-handoff to `00_orchestrator/` with routing_concern. |
| Brief contains a fact I'm uncertain about (e.g., the comparable count came back as "<3, low confidence") | Draft anyway, but add to `do_not_send_yet`: "Verify the comparable count before quoting the median to the client." |
| Draft would require me to commit the agent to something the agent shouldn't commit to without sign-off (a specific number, a specific date, a specific concession) | Add to `do_not_send_yet`. Draft proceeds. |
| Situation is sensitive AND voice file is thin (< 3 sample messages) | Produce primary + alternate. `confidence: med`. Agent picks. |

## Back-Handoffs From Me

I back-handoff to `02_property_research/` when my draft would benefit from a stat I don't have and the agent's intent isn't time-critical. The trail records it as `... 02_ → 03_ → 02_ → 03_ ...`.

I back-handoff to `04_transaction_coordinator/` only when the situation has changed in a way that affects deadline tracking (e.g., the client's reply named a date that changes a contingency timeline).

## A Note on Voice

The system is teachable in one week because it adapts to humans, not the reverse. The voice file is the seam. New agent joins → agent's voice file is captured in 30 minutes on day one (5 past messages, a closer, a salutation, a stance on emojis). After that, every draft for that agent reads like them.

That's the unlock. Without voice files, the system would have one tone, and the team would resent it.
