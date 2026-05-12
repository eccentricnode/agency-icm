# Handoff Schema

> The single contract every specialist obeys. Defined once. Referenced everywhere.

This file is the canon. Every `handoff.md` in this repo points at this schema. If a downstream specialist receives an envelope that doesn't conform, it refuses the work and back-handoffs the gap to whoever sent it.

---

## The Envelope

Every handoff between two specialists carries one structured object. The fields are:

| Field | Required | Type | Purpose |
|---|---|---|---|
| `schema_version` | yes | string | This contract's version. Currently `"1.0"`. Receivers refuse envelopes with an unknown version. |
| `case_id` | yes | string | Stable across the entire transaction. Pattern: `CASE-YYYY-NNNN` (e.g., `CASE-2026-0142`). Never reassigned. |
| `from` | yes | string | The sending specialist folder name, exact (e.g., `00_orchestrator`). |
| `to` | yes | string | The receiving specialist folder name, exact (e.g., `01_lead_qualifier`). |
| `back_to` | conditional | string \| null | Set ONLY on back-handoffs (e.g., `04_transaction_coordinator` back to `03_client_communication` for a deadline-slip email). Null otherwise. |
| `timestamp` | yes | ISO-8601 | When this envelope was produced. |
| `agent_on_deal` | yes | string | The human agent's name. Drives voice in `03_client_communication` and signatures elsewhere. |
| `payload` | yes | object | Specialist-specific contents. Shape is defined in the sending specialist's `handoff.md`. |
| `required_fields_present` | yes | array of strings | Names of the payload fields the sender confirms are populated. Receivers cross-check this against their own required list. |
| `confidence` | yes | enum: `low` \| `med` \| `high` | Sender's own honest read of how complete the payload is. Affects whether the receiver proceeds, asks for clarification, or escalates. |
| `next_action` | yes | string | One sentence the sender recommends to the receiver. The receiver may override; this is guidance, not command. |
| `trail` | yes | array of strings | Folder names this case has previously passed through, in order. Lets any specialist see the full path so far. |

---

## Canonical Example

```json
{
  "schema_version": "1.0",
  "case_id": "CASE-2026-0142",
  "from": "01_lead_qualifier",
  "to": "02_property_research",
  "back_to": null,
  "timestamp": "2026-05-11T14:22:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "intent": "buy",
    "budget": { "min": 525000, "max": 625000 },
    "timeline": "want to be in by August 2026",
    "location": ["78704", "78745"],
    "constraints": ["3+ bed", "good schools", "walkable to coffee"],
    "buyer_financing": "conventional, pre-approved"
  },
  "required_fields_present": ["intent", "budget", "timeline", "location"],
  "confidence": "high",
  "next_action": "Pull comparables and neighborhood briefs for 78704 and 78745 at 525-625k, weight schools and walkability.",
  "trail": ["00_orchestrator", "01_lead_qualifier"]
}
```

---

## Conformance Rules

A receiving specialist obeys these four rules without exception:

1. **Required fields check first.** Before reading `payload`, the receiver reads `required_fields_present` and confirms every field its own `handoff.md` lists as required is named. Missing required field → refuse, back-handoff to `from` with `confidence: low` and `next_action` naming the gap.
2. **Confidence drives behavior.** `high` → proceed. `med` → proceed but flag in the next envelope. `low` → ask, don't assume. If the receiver can't ask (autonomous run), back-handoff.
3. **Trail is append-only.** The receiver appends its own folder name when it produces its outgoing envelope. The trail is the audit log.
4. **Back-handoffs are first-class.** Setting `back_to` is not a failure; it's a normal operation. Real deals require it (a transaction coordinator routinely needs the communication specialist to draft a deadline-slip email).

---

## Human-Writable Form

Agents typing in Slack or a Claude project chat won't hand-author JSON. They write the envelope as a markdown table or a plain prose paragraph that names the fields. Specialist `examples.md` files show both forms. The receiving specialist (LLM) parses either into the structured object.

| Field | Value |
|---|---|
| case_id | CASE-2026-0142 |
| from | 01_lead_qualifier |
| to | 02_property_research |
| agent_on_deal | Diana |
| intent | buy |
| budget | 525-625k |
| timeline | in by August 2026 |
| location | 78704, 78745 |
| constraints | 3+ bed, good schools, walkable to coffee |
| confidence | high |
| next_action | Pull comparables and neighborhood briefs for 78704/78745 at 525-625k, weight schools and walkability. |

That table is sufficient. The LLM rebuilds the JSON shape on its own.

---

## Failure Modes (named)

| Failure | What it looks like | The fix |
|---|---|---|
| **Missing required field** | `required_fields_present` lacks something the receiver's `handoff.md` declares required | Back-handoff to `from` with `confidence: low`, `next_action` names the gap |
| **Stale `case_id`** | Receiver sees a `case_id` it previously closed | Back-handoff to `00_orchestrator` to confirm whether this is a re-open or a new case |
| **Voice mismatch** (specialist `03_` only) | Draft tone doesn't match `agent_on_deal`'s voice file | `03_` regenerates with the correct voice file; back-handoff only if voice file is missing entirely |
| **Deadline already passed** | `04_` receives a deadline tracker for a date in the past | Back-handoff to `00_orchestrator`; case may need human triage |
| **Confidence mismatch** | Sender said `high`, receiver finds clear gaps | Receiver downgrades `confidence` in its own outgoing envelope and notes the mismatch in `next_action` |

---

## Extending the Schema

When you add a new specialist folder (e.g., a `05_marketing_assistant`), the envelope shape does NOT change. You add the new folder name to the allowed values of `from` / `to`, define your payload shape in your new `handoff.md`, and you inherit conformance automatically.

That is the property the schema exists to provide.
