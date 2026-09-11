---
name: aviation-data-story
description: Ground a PRD, data requirements section, test plan or briefing in measured aviation data findings ("data_stories") served with the Airside Labs use-case catalogue — what ADS-B, FAA flight-plan, BTS traffic and registry data actually does when touched. Use when asked what a data source really gives you, which identifier traps to test for, how to size a feed, or to put evidence next to a use case without overstating it. Uses the Airside Labs aviation MCP tools.
---

# Aviation data stories: evidence next to the use case

A data story is a measured, dated finding from data Airside Labs holds and may
publish: its own ADS-B receiver, the FAA SWIM flight-plan feed, US BTS T-100
traffic statistics and the served entity registry. It is attached to the use
cases it says something about, and it exists to change a decision: a join key,
a validation rule, a denominator, a lag to plan for, a sampling caveat.

Needs the Airside Labs aviation tools (`mcp.airsidelabs.com/mcp`).
`get_use_case` is keyless, so every data story is reachable without an account.

## Where data stories appear

- `get_use_case(id)` returns up to three under `data_stories`, each with a
  `why_here` written for that use case.
- `search_use_cases` results carry a `data_stories` count. When two records read
  alike, take the decorated one: it comes with evidence you can put in front
  of a reviewer.
- `trace_data_lineage(message_id="ADSB")` (or `OOOI`, `MVT`, `SSM`) returns the
  data stories measured on that message type, which is the right entry when the
  question starts from the data rather than from a use case.

## The record, and which field goes where

Read the structured fields first; the narrative is last on purpose.

| Field | What it is | Where it goes in your document |
|---|---|---|
| `claim` | The finding, one sentence, scope inside it | Requirements or assumptions section, verbatim |
| `implication` | What to do about it | The design decision it drives |
| `identifiers` | Exact ids to test with (N803AL, BAW49, EVA031) | Test plan fixtures |
| `evidence` | A small table of numbers | The chart or table on the slide |
| `applies_when` | The scope: source, place, window | The footnote, always |
| `caveats`, `not_to_be_read_as` | What it does not show | Copy them; do not soften them |
| `sources` | Source id, edition, window | The citation |

Cite as: *source id, edition, window* — "Airside Labs receiver station R1,
edition 2026-09, 28 Aug to 10 Sep 2026". Never as "live data".

## The sequence

**1. Start from the decision, not the data.** "Can we key seat maps on the
flight number?" is a decision. Search the catalogue with the operational noun
(`seat map`, `tail assignment`, `flight plan`, `winds aloft`), fetch the two or
three records that carry data stories, and read the `implication` lines. If the
question starts from a feed instead ("what does SFDPS actually give me?"), go
in through `trace_data_lineage(message_id=…)`.

**2. Put the claim in the sentence that needs it.** A requirements section
that says "registration is present in 17% of FAA SWIM SFDPS messages but in
98% of flights across a day (edition 2026-09, JFK and EWR departures, 13 Aug to
10 Sep 2026); join on the flight, not the message" is a design decision with
its evidence attached. That is the whole point.

**3. Take the identifiers into the test plan.** A data story of kind
`edge_case` is a test case by another name: N803AL for a reused mark, G-EJCF
for a Mode-S address with a lost top digit, EVA031 for a zero-padded flight
number, BAW49 for four airframes under one flight number.

**4. Copy the scope, every time.** One receiver, two or three US airports,
one data edition. `applies_when` is the sentence that stops a reader
generalising a witness into a survey. If your document needs the general
rate, say that the data story does not supply it.

## Traps

- Reading a `scale` data story as an industry figure. 161 aircraft per poll is
  what one antenna sees; it is not traffic.
- Quoting a share without its denominator. Several receiver data stories carry a
  caveat that the window held 55% of the polls a full cadence would give.
- Extending a window. A data story dated to May 2026 says nothing about June.
- Treating `not_heard` or an absent data story as evidence of absence. A use
  case with no data stories is undecorated, not unillustratable.
- Calling any of it live. The data behind a data story was measured once, by a
  probe, on a dated edition. The server does not query it.

## Worked example — a dispatcher's flight-watch feature

Brief: *build the "which airframe is on this flight" panel for an OCC.*

```
search_use_cases(query="tail assignment flight plan dispatch")
   → two records carry data stories; take those.

get_use_case(<id>)
   → data stories:
     X-SWIM-REG-FILL   registration on 17.3% of messages, 98.4% of flights
                       → implication: join tail onto the flight from its
                         AH/FH/HU messages, not from whichever arrives
     X-R1-ONE-FLIGHT-MANY-TAILS  BAW49 on four B77W in 13 days; VIR105
                       on A339 and A333
                       → implication: key cabin-specific data on the
                         registration for the date, never the flight number
     X-SWIM-ZERO-PAD   EVA031 stays padded; 1.8% of identifiers
                       → implication: match on (airline, integer, suffix)

trace_data_lineage(message_id="OOOI")
   → the wheels-up data story: actual off-block-to-airborne is present for
     US departures (95.9% of domestic messages) and absent for arrivals
     from Europe → scope the milestone feature to US departures.
```

The PRD's data section is now four sentences, each with a source and a
window, and the test plan has its fixtures. Nothing in it was guessed.
