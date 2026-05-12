# Intake

> One front door. Drop anything that comes in here. The orchestrator routes from this file.

This is where every new request lands — a Zillow inquiry, a referral text, a "we want to list our house" voicemail transcript, a market-research ask from another agent on the team. You paste the raw thing in. You don't decide which specialist handles it.

---

## How To Use This

1. **Open this file in your Claude project (or wherever the system runs).**
2. **Paste the raw inbound request below the `## New Request` heading.** Don't clean it up. Don't try to summarize. Don't pre-route it.
3. **Add what you know that the message doesn't say.** One line of context if you have it — "this is from a referral from Maria's client last quarter" or "they walked into the office at 4pm yesterday." Otherwise leave blank.
4. **Hand off to `00_orchestrator/`.** The orchestrator reads this file and produces the first envelope.

You — the human agent — make one decision: drop the request in. The system makes the routing decision. This is on purpose. The newest person on your team should not need to know which specialist handles which kind of work.

---

## Template

Copy this into a new request when you need a clean slate:

```markdown
## New Request

**Received**: <date / time>
**Channel**: <Zillow lead form | text from client | voicemail transcript | email | referral | walk-in | other>
**Received by**: <agent name on the team>

### Raw Inbound

<paste the message exactly as it came in — text, screenshot transcript, voicemail words, whatever>

### Context the Agent Knows

<one or two lines of context the message itself doesn't carry, or blank>
```

---

## Worked Example

### Raw Inbound (paste-as-is)

> Hi! Saw a house on Zillow I love at 1845 Westwood Dr 78704. Wondering if it's still available and what comparable homes have sold for nearby? Budget around 550k. Need to move by end of summer. Have two kids, so schools matter. Thanks! - Sara M.

### Context the Agent Knows

Came in through the website Zillow lead form at 9:14am. No prior relationship.

### What Happens Next

The orchestrator (`00_orchestrator/`) reads this file, produces the first envelope, and routes:

- This is a **new prospect** (no prior `case_id`) → `01_lead_qualifier/` first.
- Lead qualifier confirms intent (buy), budget (~550k), timeline (end of summer), location (78704), and constraints (schools matter, 2 kids) → produces a qualified-lead envelope.
- Qualified-lead envelope is the input for `02_property_research/` on 1845 Westwood Dr and the 78704 neighborhood.
- Property research output + the agent's planned next-touch intent feeds `03_client_communication/` to draft a personalized first reply.
- When Sara responds positively and a contract gets signed, `04_transaction_coordinator/` opens the case and tracks deadlines.

You drop the request in here. The system does the rest.

---

## Why This File Exists

The brief said the orchestrator is the "front door." In practice an orchestrator is a routing layer the human agent shouldn't have to think about. The actual front door for a human is a piece of paper they paste their inbound into. That's this file.

Everything downstream depends on what gets pasted here. Garbage in, predictable garbage out. Honest in, honest out.

---

## New Request

<!-- Paste the next inbound request below this line. Delete the worked example above when you're ready to use this for real. -->

---

## Orchestrator Notes

<!-- The orchestrator writes here when it cannot process a request. Examples: raw_inbound was missing, channel was ambiguous, two routings were equally plausible. The human reads this section first thing in the morning. -->

<!-- Example: "2026-05-12 08:00 — INTAKE had no Channel line and the message body referenced 'the Wendover place' without a stable identifier. Need either a case_id or a date/agent who fielded this." -->

