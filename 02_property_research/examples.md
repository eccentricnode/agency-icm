# Examples — Property Research

Two worked research briefs.

---

## Example 1 — Buyer-Side Property + Neighborhood Brief

### Inbound Envelope (from `01_lead_qualifier/`)

> `payload.property_referenced`: "1845 Westwood Dr, 78704"
> `payload.location`: ["78704", "78745"]
> `payload.constraints`: ["good schools", "3+ beds"]
> `payload.budget`: "~550k"
> `next_action`: "Pull comparables, neighborhood character on 78704 and 78745, school zone for 1845 Westwood, flag anything an out-of-state buyer would miss."

### Brief I Produce

**Subject:** 1845 Westwood Dr, Austin TX 78704
**Window:** Closed sales May 2025 – May 2026 (6 months default + back to 12 months noted where needed)
**Comparable count:** 7 closed in 78704 within 0.6mi at 3BR matching era

#### Comparables

| Address | Sold | Beds/Baths | Sqft | Sold Price | $/sqft | Source |
|---|---|---|---|---|---|---|
| 1820 Westwood Dr | 2026-03-14 | 3/2 | 1,650 | $565,000 | $342 | MLS closed |
| 1903 Westwood Dr | 2026-02-02 | 3/2 | 1,580 | $548,000 | $347 | MLS closed |
| 1742 Westwood Dr | 2025-12-08 | 3/2.5 | 1,720 | $599,000 | $348 | MLS closed |
| 1611 Cinnamon Path | 2025-11-22 | 3/2 | 1,540 | $525,000 | $341 | MLS closed |
| 2014 Wilson St | 2025-10-19 | 3/2 | 1,690 | $580,000 | $343 | MLS closed |
| 1822 Goodrich Ave | 2025-09-30 | 3/1.5 | 1,500 | $510,000 | $340 | MLS closed |
| 1730 Westwood Dr | 2025-08-14 | 4/2 | 1,890 | $635,000 | $336 | MLS closed |

**Comparable range:** $510-635k. **Median:** $565k. **$/sqft median:** $343.

#### 78704 Neighborhood Character

**What it is.** Pre-1990 housing stock (mostly 1940s-1960s ranch and bungalow), high walkability, dense small businesses on South Lamar / South First / South Congress. Buyer profile lately skews professional couples and small families; some teardown / scrape-and-build activity but Westwood corridor has held against it.

**What's distinctive.** 78704 is the "South Austin" identity bucket — Zilker, Bouldin, Travis Heights, Barton Hills, South Lamar corridor. The Westwood/Goodrich pocket is quieter than Bouldin proper, with the same school zone (Zilker Elementary).

**What an out-of-state buyer would miss.** South Lamar weekend traffic from May through Sept is real — anything within 4 blocks of Lamar carries a noise/traffic delta. Foundation movement on 1940s-1960s pier-and-beam in 78704 is common and survivable; a buyer should plan to inspect specifically for foundation and to budget $3-8k for routine leveling within 5 years of purchase.

#### School Zone

- **Elementary**: Zilker Elementary — GreatSchools 8/10, NeighborhoodScout B+
- **Middle**: O. Henry Middle — GreatSchools 7/10
- **High**: Austin High — GreatSchools 7/10

(Source: GreatSchools ratings as of 2026-Q1; AISD attendance zones verified at austinisd.org.)

#### The Buyer Should Know

A 3BR in this comparable set under 550k will likely be ≤1,580 sqft or need foundation work; a buyer at 550k stretching to 575-580k opens the 1,650-1,720 sqft band on Westwood proper.

### Envelope Wrapping the Brief

```json
{
  "case_id": "CASE-2026-0143",
  "from": "02_property_research",
  "to": "03_client_communication",
  "back_to": null,
  "timestamp": "2026-05-11T10:14:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "brief_subject": "1845 Westwood Dr, Austin TX 78704",
    "comparable_count": 7,
    "comparable_range": "$510-635k",
    "comparable_median": 565000,
    "neighborhood": "78704 — Westwood corridor",
    "school_zone": "Zilker / O. Henry / Austin High",
    "agent_pull_quotes": [
      "Comparable range $510-635k; median $565; a 3BR under 550k is likely ≤1,580 sqft or needs foundation work.",
      "South Lamar weekend traffic adds noise within 4 blocks; foundation movement common on 1940s-1960s pier-and-beam, budget $3-8k.",
      "1845 Westwood is squarely in the Westwood pocket — same school zone as Bouldin proper, quieter."
    ],
    "trec_form_implied": "TREC 20-18 (One to Four Family Residential — Resale)",
    "sources_named": ["MLS closed-sale", "GreatSchools 2026-Q1", "AISD attendance"]
  },
  "required_fields_present": ["brief_subject", "comparable_count", "agent_pull_quotes", "sources_named"],
  "confidence": "high",
  "next_action": "Draft Sara's first reply using these three pull quotes; offer a showing for 1845 Westwood and one comparable.",
  "trail": ["00_orchestrator", "01_lead_qualifier", "02_property_research"]
}
```

---

## Example 2 — Sell-Side Market Read for a Vague Seller

### Inbound

> `payload.location`: ["78745"]
> `payload.constraints`: ["3BR, bought 2022"]
> `next_action`: "Sketch a sell-side market read for Mike's place in 78745 to inform a 15-minute call this week."

### Brief I Produce

**Subject:** 78745 sell-side market read, 3BR
**Window:** Closed sales Mar-May 2026

| Metric | Value | Source |
|---|---|---|
| Median sold price 3BR 78745 | $478,000 | MLS closed Mar-May 2026 |
| Median DOM | 31 days | MLS |
| Months-of-inventory | 3.4 | TXR market data 2026-Q1 |
| 2022 buy-price for 78745 3BR (Mike's purchase era) | $445,000 median | MLS closed 2022 |

**The Seller Should Know.** Net of selling costs (6% commission + closing ~1.5%) at $478k median: roughly $442k net. If Mike bought at the 2022 median ($445k), he's roughly even before factoring any improvements he made. The call should establish whether his motivation is move-up, cash-out, or simply curiosity — none of those are bad, but the right framing changes everything.

```json
{
  "case_id": "CASE-2026-0144",
  "from": "02_property_research",
  "to": "03_client_communication",
  "back_to": null,
  "timestamp": "2026-05-11T18:02:00-05:00",
  "agent_on_deal": "Diana",
  "payload": {
    "brief_subject": "78745 sell-side market read, 3BR",
    "median_sold_3br": 478000,
    "median_dom": 31,
    "months_of_inventory": 3.4,
    "implied_net_at_median": 442000,
    "agent_pull_quotes": [
      "Median 3BR sold in 78745 last 90 days: $478k, 31 DOM, 3.4 MOI.",
      "Net of typical selling costs: ~$442k.",
      "If Mike bought near 2022 median ($445k), he's roughly even pre-improvements."
    ],
    "sources_named": ["MLS closed Mar-May 2026", "TXR market data 2026-Q1"]
  },
  "required_fields_present": ["brief_subject", "agent_pull_quotes", "sources_named"],
  "confidence": "med",
  "next_action": "Drive a 15-minute call agenda around move-up vs. cash-out vs. curiosity; do not propose a listing price on the call.",
  "trail": ["00_orchestrator", "01_lead_qualifier", "02_property_research"]
}
```

---

## What These Examples Show

- I produce three things every brief carries: data table, neighborhood/market context, and an out-of-state-buyer-would-miss note.
- `agent_pull_quotes` are what `03_client_communication/` turns into prose for the client. The structure makes their job easier.
- `sources_named` is part of the conformance check — `03_` refuses to act if I send a brief with no sources named.
