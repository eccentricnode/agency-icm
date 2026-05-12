# agency-icm

> An AI operating system for a small real estate team. Five specialist folders, one envelope schema, and one front door — so a newest agent is operational in a day.

---

## It's 11pm

Your newest agent just texted: which document needs to go where for the inspection objection that's due Friday? You're putting the kids to bed. You answer, and as you type you realize this same exchange happened on Tuesday with a different deal and a different document.

Your team is four people doing 70 transactions a year. Everyone is good at their part. Everyone is also the only one who knows their part. The systems live in your head and three Google Docs that nobody updates after Monday.

You don't want another platform. You want a system you can teach your whole team to use in one week.

This is that.

---

## What You'll Get From This

You'll get a working folder structure your team can drop into a Claude project (or any LLM context) on Monday morning. By Friday you'll know whether it fits how your team actually works. By the end of the quarter you'll be teaching the next hire from the same folder, not from your head.

You won't get software. You won't get a SaaS platform you'll outgrow in a year. You'll get something simpler and harder to copy: a contract between five specialists that doesn't drift because it lives in files, not in heads.

---

## What This Is For

A boutique real estate team — typically 3 to 6 people doing 50 to 100 transactions a year — that's:

→ Running on three Google Docs and the principal's memory
→ Watching lead response depend on whoever-saw-the-notification-first
→ Re-doing property research from scratch every time
→ Getting 11pm Slacks from the newest hire about which document goes where

If that sounds like your week, this is for you. The brief that started this build came from Diana, who runs a 4-person boutique team in Austin, Texas. You can adapt this for your locale — see the *Where to go next* section.

---

## How It Works (One Paragraph)

Every new request lands in `INTAKE.md` at the root. The orchestrator (`00_orchestrator/`) reads it and routes it to one of four specialists — lead qualifier, property research, client communication, or transaction coordinator. Each specialist reads its own four context files plus the **envelope** it received, does its work, and produces an outgoing envelope for whichever specialist comes next. The envelope shape lives in **one** file at the root: `HANDOFF_SCHEMA.md`. That's the contract. When the contract holds, the team holds.

```
        ┌─────────────┐
        │  INTAKE.md  │ ← human pastes raw inbound here
        └──────┬──────┘
               ▼
   ┌───────────────────────┐
   │   00_orchestrator/    │ ← routes; produces first envelope
   └─────┬─────┬─────┬─────┘
         ▼     ▼     ▼
   ┌─────────┐ ┌─────────────┐ ┌──────────────┐
   │   01_   │ │     02_     │ │     03_      │
   │  lead   │ │  property   │ │   client     │
   │qualifier│ │  research   │ │communication │
   └────┬────┘ └──────┬──────┘ └──────┬───────┘
        └────────┬────┴────────────────┘
                 ▼
         ┌───────────────────────┐
         │   04_transaction_     │
         │   coordinator        │ ← takes over once under contract
         └───────────┬───────────┘
                     │
                     └──── back-handoff to 03_ when client message needed
```

The trail of every case (which specialists touched it, in what order) is recorded in every envelope. The trail is the audit log.

---

## The Three Principles

Three rules every specialist obeys. They sound abstract; they earn their keep on day fourteen, when someone tries to change one thing and three other things keep working.

→ **One fact, one location.** The envelope schema lives in `HANDOFF_SCHEMA.md` — once. Every specialist references it. Nobody re-declares it. Drift in the schema is drift in the team, so we don't allow it.

→ **New sessions start clean.** Each specialist reads its own four files (`identity.md`, `rules.md`, `examples.md`, `handoff.md`) plus the envelope it received. No implicit context. No "you should know that…". The LLM running each step starts cold and finishes ready.

→ **Same quality for everyone.** Your newest agent and your most senior agent get the same artifact, because context lives in files not in heads. That's the test. If a stranger to your team cannot pick up a case from the trail and continue it, the architecture is failing.

---

## Why Not Just…

A few honest objections, answered.

→ **"Why not just use a SaaS CRM like Follow Up Boss?"** A CRM is fine for storing contacts and tracking pipelines. It's not where your team's *judgment* lives. The files in this repo encode the judgment — what to capture from a lead, what makes a research brief useful, how to draft in your voice. A CRM and this repo are not the same thing; you'll likely run both.

→ **"Why not write actual code instead of markdown?"** Because the folders ARE the system. The LLM is the runtime. Code adds maintenance overhead with no leverage — every change has to be debugged, deployed, and re-taught to the team. Markdown is debuggable by reading.

→ **"Why not have specialists work in parallel?"** Because real estate handoffs are sequential — you qualify before you research, you research before you reply, you sign a contract before you track deadlines. Parallel fanout is a real pattern, but for a 4-person team it adds coordination cost without proportional benefit. If your team grows past 15 people, revisit this.

→ **"Why one envelope schema instead of bespoke contracts per pair?"** Because pairwise contracts drift. Five folders = ten directed pairs = ten chances for a spec to slip. One canonical envelope is one chance.

---

## Try The Smallest Version

You don't need all five folders to feel how this works. Try this first.

Make a directory with just two folders and one schema file:

```
demo/
├── HANDOFF_SCHEMA.md     ← copy from this repo
├── INTAKE.md             ← copy and clear out the worked example
├── 00_orchestrator/
│   └── identity.md       ← I am the router. I read INTAKE.md and produce an envelope to 01_.
└── 01_lead_qualifier/
    └── identity.md       ← I qualify leads. I read the envelope and produce a qualified-lead envelope.
```

Drop a fake inbound into `INTAKE.md`:

```markdown
## New Request

**Received**: 2026-05-12
**Channel**: text

### Raw Inbound

Hey, I'm Mike. Saw a listing on Realtor.com at 1234 Main St. Budget around 600k. Need to move in two months.
```

Hand the whole `demo/` folder to Claude (or any LLM) and ask: *"Read INTAKE.md, then act as 00_orchestrator and produce the envelope to 01_lead_qualifier. Then act as 01_lead_qualifier and produce the qualified-lead envelope. Show both envelopes."*

You'll see the shape work. The envelope passes data from one specialist to the next. The schema in `HANDOFF_SCHEMA.md` keeps them aligned. You've just done in two folders what the full system does in five.

When that feels real, come back here and use the full repo.

---

## Onboarding a New Team Member

When you hire a fifth agent, here's their first day. It takes about an hour.

1. **Read `README.md`** (this file) — 10 minutes. Get the architecture frame.
2. **Read `INTAKE.md`** and `HANDOFF_SCHEMA.md` — 15 minutes. Get the contract and the front door.
3. **Walk one case from beginning to end** — pick a recently closed deal. Read the trail. Open each specialist folder in the order the trail names. Read that folder's `examples.md`. 30 minutes.
4. **Capture their voice** for `03_client_communication/`. Have them paste 5 of their recent past client messages into a file you'll save as `voice/<their-name>.md`. Add their closer, their salutation, and a sentence on their stance on emojis. 15 minutes.

That's the day. They're operational tomorrow.

The trick is that the system adapts to them through the voice file. The first message they send to a client through this system will sound like them, because the system read their words first.

---

## A Typical Request, End to End

Day 1, 9:14am — a Zillow lead form fills out. Sara M., asking about 1845 Westwood Dr in 78704. Diana sees the notification and pastes the message into `INTAKE.md`.

→ `00_orchestrator/` reads the intake, opens **CASE-2026-0143**, routes to `01_lead_qualifier/` with `confidence: high`.

→ `01_lead_qualifier/` reads Sara's words, captures intent (buy), budget (~550k), timeline (end of summer), location (78704), constraint (good schools, 2 kids → 3+ beds implied). Notes the gaps (pre-approval status, financing type, current housing). Sends to `02_property_research/`.

→ `02_property_research/` pulls 7 closed comparables on Westwood in the last six months, the school zone (Zilker / O. Henry / Austin High), and the one thing an out-of-state buyer would miss (pier-and-beam foundation, 1940s-1960s era, plan to budget $3-8k for routine leveling). Sends to `03_client_communication/`.

→ `03_client_communication/` loads Diana's voice file, drafts a reply that quotes the comparable range, names the school zone, mentions the foundation note, and proposes a Saturday showing of three properties on the same trip. Diana reads, sends. Trail so far: `00 → 01 → 02 → 03`.

Sara replies, says she's pre-approved with Chase, wants to see the houses. Saturday happens. By Monday Sara has signed a contract on 1845 Westwood.

→ `00_orchestrator/` opens the transaction phase, routes to `04_transaction_coordinator/`.

→ `04_transaction_coordinator/` confirms the effective date (the date the final signature was *communicated in writing* — Day 2, not Day 1, because the listing agent emailed the executed PDF the morning after signing). Logs earnest money deadline (3 business days), option period end (10 days from effective), title commitment delivery, appraisal contingency, closing.

Day 8: appraisal report is late. `04_` back-handoffs to `03_`. `03_` drafts Diana a heads-up email to Sara naming the date and the extension amendment as a planned response. Diana sends. `04_` pre-stages the extension amendment. Trail: `… → 04 → 03 → 04`.

Day 35: closing.

That's the whole thing. Five folders. One contract. One audit trail. No memory of who-was-supposed-to-do-what — the trail says.

---

## Setup Before You Drop This Into A Claude Project

Three things:

1. **Clone this repo.** That's it. The folder structure IS the system.
2. **Create voice files for your team.** The `voice/` folder exists; `voice/EXAMPLE_voice_diana.md` ships with the repo so you see the shape. Add a `voice/<agent_name>.md` for each person on your team. Skip this and `03_client_communication/` will refuse to draft. The file is short — 5 past messages, a closer, a salutation, a stance on emojis. (Per-agent files are git-ignored so private writing samples don't end up in your repo.)
3. **Pick a place to keep case state.** A `cases/` folder with one file per `case_id` is the simplest convention; Notion or a shared Google Doc also works. The repo doesn't force a choice, but each case needs *somewhere* to live so the trail is real. A convention worth committing to: `cases/CASE-YYYY-NNNN.md` with the latest envelope at the top and prior envelopes below. The orchestrator scans `cases/` to discover prior `case_id`s.

That's everything. No software to install. No accounts to create.

---

## Where To Go Next

Pick the path that matches what you need.

→ **You want to use this on your team this week.** Start with the smallest version above. Then read each folder's `identity.md` and `rules.md`. Capture voice files for everyone. Run one real case through it on Friday. Don't try to onboard everything at once.

→ **You want to adapt this for a different locale (not Austin TX).** The Texas-specific bits live in `04_transaction_coordinator/rules.md` and `02_property_research/examples.md`. The architecture is locale-agnostic; replace TREC form numbers, contingency standards, and the "effective date" quirk with your state's equivalents.

→ **You want to add a sixth specialist (marketing assistant, listings coordinator, etc.).** Make the folder. Write the four files. Add the new folder name to the allowed `to` values in `HANDOFF_SCHEMA.md`. Decide which existing specialists hand off to it and edit their `handoff.md` files. The envelope shape does not change. That's the whole point.

→ **You want to teach this to another team.** Read `00_orchestrator/identity.md`. Read `04_transaction_coordinator/handoff.md` for the back-handoff pattern. Those two files are the architecture's load-bearing walls; everything else inherits from them.

---

## Go Deeper

Each specialist's full description lives in its folder.

| Folder | Read in this order |
|---|---|
| `00_orchestrator/` | identity → rules → examples → handoff |
| `01_lead_qualifier/` | identity → rules → examples → handoff |
| `02_property_research/` | identity → rules → examples → handoff |
| `03_client_communication/` | identity → rules → examples → handoff |
| `04_transaction_coordinator/` | identity → rules → examples → handoff (the back-handoff section is the load-bearing one) |

Root files:

| File | Purpose |
|---|---|
| `INTAKE.md` | The one front door for every new request |
| `HANDOFF_SCHEMA.md` | The envelope contract every specialist obeys |
| `LICENSE` | MIT |

---

*Built for [Jake van Clief's CliffNotes Weekly Competition #4 — The Agency](https://www.skool.com/cliffnotes), May 2026. Architecture pattern: Interpretable Context Methodology (ICM), Jake-pure. License: MIT.*
