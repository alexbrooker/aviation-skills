---
name: aviation-opportunity-map
description: Build an aviation AI opportunity map, briefing deck or PRD grounded in a real use-case catalogue — for an airport, airline, ground handler, ANSP or MRO. Use when asked what AI use cases exist for an aviation role or organisation, what data each needs and how fresh, what benefits downstream if one prediction improves, or to turn a workshop or discovery conversation into a shortlist. Uses the Airside Labs aviation MCP tools.
---

# Aviation AI opportunity map

Turns "where could AI help us" into a shortlist you can defend: real use cases
attached to real roles, the data each needs with its cadence and source system,
and what benefits downstream when one of them improves.

Needs the Airside Labs aviation tools (`mcp.airsidelabs.com/mcp`).
`use_case_landscape`, `get_use_case`, `submit_suggestion` and `feedback_status`
work with **no account at all** (a per-address daily cap applies); the rest need
a key. So the scoping call in step 1 and the full record in step 4 cost you
nothing to try.

## The sequence, and why it is this order

**1. Scope before you search.** Call `use_case_landscape` first, grouped by
`org_type` or by `role` within one org type. It tells you what the catalogue
covers and the exact spellings the filters accept, so you never guess a filter
value. It also returns how many use cases in scope carry an EASA screen, which
sets expectations before anyone asks.

Counts describe the catalogue, not the industry. A type with many use cases is
one the corpus describes in detail, not one that adopts more AI. Never present a
count as market evidence.

**2. Search operational nouns, not capability labels.** `search_use_cases` is
BM25 over the corpus, and the corpus vocabulary is operational. "stand
allocation", "baggage misconnect", "de-icing", "turnaround", "taxi-out" work.
"digital transformation", "shared situational awareness", "operational
excellence" return noise or nothing.

Terms are OR-ed, so a long query returns partial matches ranked. `score` is
relative within one query and means nothing across queries.

**3. Read the results with a filter in your head.** The catalogue mixes richly
specified use cases with one-line generic ones, and nothing in the response
marks which is which. A record reading *"Optimize X using predictive analytics
and machine learning models"* will not carry an argument. One naming actual
operational objects — TOBT, in-block time, minimum turn time, stand type — will.
Prefer the second, and if a search returns only the first kind, your query was
probably too abstract: go back to step 2 with a narrower noun.

**4. Fetch few, and fetch deliberately.** `get_use_case` returns the full record:
the role with its job description, every data requirement with its cadence class
and source system, the EASA screen with a rationale and confidence, and page-cited
framework rows. Pull the two or three an argument actually needs. Walking the
catalogue record by record is the wrong tool and burns your allowance — use the
aggregates instead.

**4b. Read the exhibits before you write the slide.** A full record may
carry up to three `exhibits`: measured, dated findings from Airside Labs' own
ADS-B receiver, the FAA flight-plan feed, BTS traffic statistics or the served
entity registry, each with a `why_here` written for that use case. Put the
`claim` on the slide as the finding, the `evidence` table as the chart, and the
source id and window as the footnote; copy `caveats` and `not_to_be_read_as`
rather than softening them. Search results carry an `exhibits` count, so when
two records read alike take the decorated one. One receiver and three US
airports are a witness, not a survey: never extend an exhibit past its
`applies_when`, and never call it live. The `aviation-data-story` skill covers
this in depth.

**5. The step most people miss: reverse lineage.** `data_requirements(query="…")`
searches the *inputs*, not the use-case text, and returns everything that CONSUMES
data matching your query. Ask `query="turnaround duration TOBT"` and you get every
use case downstream of a better turnaround estimate, across every organisation.

This is the slide an executive actually buys — "improve this one feed and here is
who benefits" — and it is the single most under-used thing in the toolset.

**This step needs a key.** Steps 1 and 4 are keyless, so you can scope and pull
records before anyone signs up, but the reverse lineage is a paid tool. If you
are working without one, say so at this point rather than quietly delivering a
weaker deck: the shortlist from steps 1-4 is genuinely useful, and the
"who benefits downstream" argument is the thing that needs an account.

**6. Count feeds on families, never on titles.** The same aggregate returns
`family_mix`, which collapses ~8,000 raw requirement titles into ~29 canonical
families. "Weather Data", "Weather and Environmental Data" and "Environmental
Conditions" are one feed. Counting raw titles overstates a feed inventory by
roughly 2-3x. About 24% stay in family `other`, which is honest heterogeneity,
not a gap.

**7. Optional, and strong for a technical audience:** `trace_data_lineage` turns
a data requirement into the standard messages that would carry it (MVT, LDM, DPI,
METAR…), and `trace_workflow` shows what runs before and after a use case and who
hands off to whom. `trace_workflow` is the operational dependency; the reverse
lineage in step 5 is a shared-feed correlation. Both are useful and they are not
the same claim.

## Worked example — turnaround delay at a mid-size airport

The brief: *"Where can AI reduce turnaround delay?"* for an airport COO. This is
the actual sequence, with what each call is for.

```
use_case_landscape(group_by="role", org_type="Ground Handling")
   → scope. Note if a domain looks thin: ground-ops work is spread across
     Ground Handling, Aviation Services, Airline and Airport, so one slice
     understates it. Search across org types rather than trusting one count.

search_use_cases(query="aircraft turnaround delay prediction stand")
   → candidates. Keep the specific ones; discard "Optimize turnaround times
     using predictive analytics".

get_use_case(11539)
   → the anchor record: predicting actual turn-round duration, producing
     auto-calculated TOBTs that beat static minimum-turn tables. Gives you
     four data requirements with cadence and source, and a 1B/H4 screen.

data_requirements(query="turnaround duration TOBT")
   → the payoff. Hundreds of consuming use cases, and the top ones span four
     organisations: the handler's duty manager, the airline's ops controller,
     the airport's A-CDM coordinator and stand planner, the ANSP's tower
     supervisor, the de-icing coordinator.
```

**The deck writes itself from that fourth call.** One prediction, four
organisations, each with a named role and a stated data need. That is the
argument; the earlier calls are the evidence for it.

## Writing it up

State the provenance. Every response carries a `provenance` block: `corpus` is
Airside Labs' catalogue (use it in your analysis; do not redistribute it as a
dataset), `derived` is their assessment over it, `framework:easa` is public
regulatory text you may quote with its `cp_ref` page.

Say what the catalogue cannot tell you, because a reader will assume otherwise:
it describes plausible, well-formed use cases for a role and says **nothing about
adoption** — not whether anyone runs such a system, not which vendors sell one,
not whether any is certified. If someone needs "who else is doing this", that is
a different research task and the honest answer is that this does not answer it.

An empty result is a real answer. If the catalogue has nothing on a phrasing,
say so and file it with `report_unmet_need` (`gap_kind: no_use_case`) — that is
how the gap gets closed for the next person.

## Traps

- Guessing a filter value instead of reading it from `use_case_landscape`.
- Searching capability language. It returns noise and reads as a thin catalogue.
- Quoting a use-case count as evidence of industry adoption.
- Counting raw requirement titles as distinct feeds.
- Fetching dozens of full records when an aggregate answers the question.
- Presenting a generic one-line use case as though it were a specified one.
- Quoting an exhibit as an industry statistic or as live data; it is one
  sensor or one feed, on one dated edition, and its scope is in `applies_when`.
