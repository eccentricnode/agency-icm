# Examples — Lead Qualifier

Two qualified-lead envelopes from real-ish raw inbounds.

---

## Example 1 — The Zillow Buyer

### Inbound Envelope (from `00_orchestrator/`)

> `payload.raw_inbound`: "Saw a house on Zillow I love at 1845 Westwood Dr 78704. Wondering if it's still available and what comparable homes have sold for nearby? Budget around 550k. Need to move by end of summer. Have two kids, so schools matter. Thanks! - Sara M."

### What I Captured

| Field | Value | Source |
|---|---|---|
| intent | buy | inferred from Zillow lead context + property reference |
| budget | "around 550k" | prospect's words |
| timeline | "end of summer" | prospect's words (interpret as Aug-Sep 2026) |
| location | 78704 (named) | prospect's words |
| constraints | schools important; 2 kids | prospect's words |
| prior_relationship | none — Zillow lead form | channel context |

Gaps:
- Pre-approval status not stated.
- Financing type not stated.
- Min beds/baths not stated (2 kids implies 3+ beds but not stated).
- Move-from address / current housing situation not stated.

### Envelope I Produce — Human-Writable Form

| Field | Value |
|---|---|
| case_id | CASE-2026-0143 |
| from | 01_lead_qualifier |
| to | 03_client_communication |
| back_to | null |
| timestamp | 2026-05-11T09:22:00-05:00 |
| agent_on_deal | Diana |
| confidence | high |
| next_action | Draft first reply: confirm 1845 Westwood availability, propose a showing, ask about pre-approval and current housing, name 78704 + 78745 as likely zones given schools criterion. |
| trail | 00_orchestrator → 01_lead_qualifier |

### JSON Form

```json
{
  "case_id": "CASE-2026-0143",
  "from": "01_lead_qualifier",
  "to": "03_client_communication",
  "back_to": null,
  "timestamp": "2026-05-11T09:22:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "prospect_name": "Sara M.",
    "intent": "buy",
    "intent_source": "inferred from Zillow lead + property reference",
    "budget": "~550k (prospect's word: 'around 550k')",
    "timeline": "end of summer — Aug-Sep 2026",
    "location": ["78704"],
    "location_inferred": ["78745"],
    "constraints": ["good schools", "2 kids (implies 3+ beds)"],
    "prior_relationship": "none — Zillow lead",
    "property_referenced": "1845 Westwood Dr, 78704",
    "gaps": ["pre-approval status", "financing type", "min beds/baths explicit", "current housing situation"],
    "inference_notes": "Inferred buy from Zillow lead form context; inferred 78745 as likely also-acceptable based on similar school options and price point."
  },
  "required_fields_present": ["prospect_name", "intent", "budget", "timeline", "location", "constraints", "prior_relationship", "gaps"],
  "confidence": "high",
  "next_action": "Draft first reply: confirm 1845 Westwood availability, propose a showing, ask about pre-approval and current housing, name 78704 + 78745 as likely zones given schools criterion.",
  "trail": ["00_orchestrator", "01_lead_qualifier"]
}
```

---

## Example 2 — The Vague Seller

### Inbound Envelope (from `00_orchestrator/`)

> `payload.raw_inbound`: "Hey Diana, I've been thinking about maybe putting my place on the market sometime this year. Just wanted to see what you think. Talk soon? - Mike"
>
> `payload.channel`: "text"
> `payload.received_at`: "2026-05-11T17:33:00-05:00"
> `payload.prospect_name`: "Mike"
> `payload.prior_relationship`: "(Diana's added context) Past buyer-side client 2022, bought 3BR in 78745."

### What I Captured

| Field | Value | Source |
|---|---|---|
| intent | sell | prospect's words ("on the market") |
| budget | unknown (sell-side; valuation needed) | n/a |
| timeline | "sometime this year" — vague | prospect's words |
| location | 78745, 3BR (from prior relationship) | prior case data |
| constraints | unknown | n/a |
| prior_relationship | past buyer-side client 2022 | agent's note |

Gaps:
- Specific timeline (Q3? Q4? next month?).
- Whether he wants to sell-and-stay (needs a buy-side too) or sell-and-leave.
- Whether the property has changed since 2022.
- His read on current value (or the gap between his read and market).

### Envelope I Produce — JSON Form

```json
{
  "case_id": "CASE-2026-0144",
  "from": "01_lead_qualifier",
  "to": "03_client_communication",
  "back_to": null,
  "timestamp": "2026-05-11T17:40:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "prospect_name": "Mike",
    "intent": "sell",
    "intent_source": "prospect's words",
    "budget": "unknown — sell-side, valuation needed",
    "timeline": "'sometime this year' — vague",
    "location": ["78745"],
    "constraints": [],
    "prior_relationship": "past buyer-side client 2022 — 3BR in 78745",
    "gaps": ["timeline specificity", "sell-and-stay vs sell-and-leave", "property condition since 2022", "his read on current value"],
    "inference_notes": "Buyer-side relationship from 2022 means warm rapport; Diana can be direct in first reply without losing tone."
  },
  "required_fields_present": ["prospect_name", "intent", "budget", "timeline", "location", "constraints", "prior_relationship", "gaps"],
  "confidence": "med",
  "next_action": "Draft warm short reply: acknowledge, propose 15-minute call this week, ask the timeline-tightening + sell-and-stay-vs-leave questions on the call rather than in text.",
  "trail": ["00_orchestrator", "01_lead_qualifier"]
}
```

---

## What These Examples Show

- I quote the prospect's exact words when I can. The downstream draft will reuse those words.
- I name gaps explicitly. The first reply (or the first call) will close them.
- `confidence: med` is honest — Mike's intent is clear but his timeline isn't, and that affects the right tone for the reply.
- Inferred fields are always flagged; the agent reading downstream knows what the prospect said vs. what I read into it.
