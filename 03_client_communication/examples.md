# Examples — Client Communication

Three drafts. The first is a first-reply to a Zillow lead (routine). The second is a competing-offer scenario (sensitive — two alternates). The third is a back-handoff from `04_` for a deadline-slip email.

---

## Example 1 — First Reply to Zillow Lead

### Inbound

From `02_property_research/`. Agent is Diana. Three pull quotes ready. Sara M. asked about 1845 Westwood Dr.

### Diana's Voice File (snippets the agent provided)

- Closer: "talk soon — Diana"
- Salutation: "Hi <first name> —"
- Contractions: heavy.
- Emoji: rare. Never on first contact.
- Pattern: short paragraphs. Specific, not generic. Always ends with a clear next step.

### Draft (Primary)

```
Hi Sara —

Thanks for the note about 1845 Westwood. It's still active as of this morning. The Westwood pocket of 78704 is a nice find for you — same school zone as Bouldin proper (Zilker / O. Henry / Austin High), and quieter.

Comparable 3-bed homes have closed in the $510k to $635k range in the last six months on that block. Median around $565k. At your budget around 550k, you're solidly in the running for 1845 specifically or its neighbors.

One thing worth knowing before we walk it: the 1940s-1960s homes on Westwood are pier-and-beam, and most of them need routine foundation leveling at some point — usually $3-8k. Worth budgeting for.

Want to see it this weekend? I can pull a couple of other Westwood-corridor places on the same trip so you have a feel for the range.

talk soon —
Diana
```

### Why This Works

- Uses Diana's exact closer.
- Quotes the brief's three pull quotes naturally — no smoothing into "very competitive market."
- Includes the foundation note (the out-of-state-buyer-would-miss item) because it's the value-add the brief produced.
- Names a specific next step (this weekend, multiple properties on one trip).
- No AI-tells.

### Envelope I Produce

```json
{
  "schema_version": "1.0",
  "case_id": "CASE-2026-0143",
  "from": "03_client_communication",
  "to": "00_orchestrator",
  "back_to": null,
  "timestamp": "2026-05-11T10:32:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "channel": "email",
    "draft_primary": "...full text above...",
    "alternates": [],
    "do_not_send_yet": [],
    "agent_review_notes": "Mentions foundation, which the prospect didn't ask about — Diana should confirm she wants that in the first message."
  },
  "required_fields_present": ["channel", "draft_primary"],
  "confidence": "high",
  "next_action": "Send after Diana's review; reply expected same day.",
  "trail": ["00_orchestrator", "01_lead_qualifier", "02_property_research", "03_client_communication"]
}
```

---

## Example 2 — Competing-Offer Reply (Sensitive)

### Inbound

Buyer client is one of three offers on a listing in 78745. Listing agent set a 24-hour response deadline. `02_` pulled comparables; the property is priced just under the recent-comparable median. Agent on deal is Diana.

### Draft (Primary — Recommended)

```
Hi Sara —

Quick heads-up: the listing agent confirmed three offers on Wendover, and they're asking for highest-and-best by 6pm tomorrow.

My read: the home is priced right under the comparable median for the block ($478k vs median $485k from the last six months). A clean offer at or slightly above list, with a short option period and standard contingencies, will compete with or beat strategic-low offers. I'd avoid escalation clauses — in Austin they signal inexperience to listing agents and they don't usually buy you what you'd think.

If you want to write a short personal note about why the house works for your family, I'll include it with our offer. Those help more than people expect.

Want to chat for 15 minutes today to finalize the number?

talk soon —
Diana
```

### Draft (Alternate — More Cautious)

```
Hi Sara —

Wanted to update you on Wendover before you read about it elsewhere: the listing agent confirmed three offers and is asking for highest-and-best by 6pm tomorrow.

Before we move, I want to make sure we have a clear ceiling and a clean strategy. Comparable 3-beds on that block have closed in a tight $470-495k range over the last six months — the asking price is right at the comp midline.

Some options to talk through: at-list with a tight option period, slightly above with standard contingencies, or stepping aside if the right number isn't in the cards.

Can we talk for 15 minutes today? No rush on the offer side until we've aligned.

talk soon —
Diana
```

### Envelope

```json
{
  "schema_version": "1.0",
  "case_id": "CASE-2026-0143",
  "from": "03_client_communication",
  "to": "00_orchestrator",
  "back_to": null,
  "timestamp": "2026-05-12T08:14:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "channel": "email",
    "draft_primary": "...full text above (recommended)...",
    "alternates": ["...full text above (cautious)..."],
    "do_not_send_yet": [],
    "agent_review_notes": "Both drafts avoid escalation-clause language explicitly. Primary nudges toward action; alternate slows things down. Diana picks based on Sara's risk tolerance."
  },
  "required_fields_present": ["channel", "draft_primary", "alternates"],
  "confidence": "med",
  "next_action": "Diana picks primary or alternate. Send same day; client's reply may come in within an hour.",
  "trail": ["00_orchestrator", "02_property_research", "03_client_communication"]
}
```

---

## Example 3 — Deadline-Slip Email (Back-Handoff from `04_`)

### Inbound

`04_transaction_coordinator/` back-handoffs because the appraisal report is 2 days late and the appraisal-contingency deadline is in 3 days. The buyer hasn't been told yet. Agent is Diana.

### Draft

```
Hi Sara —

Quick update on the appraisal: the appraiser's report was due Monday and we still don't have it. I've followed up with the lender twice this morning.

Where this matters: the appraisal contingency runs out Friday. If the report doesn't land by Thursday EOD, we'll send a short extension amendment to push the contingency by a week. That's normal in this market and doesn't put the deal at risk on its own.

I'll know more by 5pm today. If the report lands, you'll hear from me. If it doesn't, I'll have the amendment ready and just need a quick OK from you.

talk soon —
Diana
```

### Envelope

```json
{
  "schema_version": "1.0",
  "case_id": "CASE-2026-0119",
  "from": "03_client_communication",
  "to": "04_transaction_coordinator",
  "back_to": "04_transaction_coordinator",
  "timestamp": "2026-05-13T09:48:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "channel": "email",
    "draft_primary": "...full text above...",
    "alternates": [],
    "do_not_send_yet": [],
    "agent_review_notes": "Once Diana sends, `04_` should pre-stage the extension amendment so it's ready for signature if the report doesn't land Thursday."
  },
  "required_fields_present": ["channel", "draft_primary"],
  "confidence": "high",
  "next_action": "Diana sends. `04_` pre-stages the extension amendment.",
  "trail": ["00_orchestrator", "04_transaction_coordinator", "03_client_communication", "04_transaction_coordinator"]
}
```

Note the trail: 04 → 03 → 04. The back-handoff is visible.

---

## What These Examples Show

- Drafts read like Diana — short paragraphs, "talk soon", contractions, no fluff.
- Competing-offer draft explicitly refuses the escalation-clause path (a Texas-specific move that signals craft).
- Back-handoff from `04_` is treated as a normal operation, not an emergency.
- `agent_review_notes` gives the agent the meta-commentary so they don't have to reconstruct my reasoning.
