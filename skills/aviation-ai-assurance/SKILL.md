---
name: aviation-ai-assurance
description: Screen an aviation AI use case against the EASA AI-level and hazard-class framework, and write the assurance section of a PRD, business case or regulatory conversation. Use when asked how an aviation AI system would be classified, what assurance level it implies, whether a technique is acceptable at that level, or what EASA's proposed guidance says about a use case. Uses the Airside Labs aviation MCP tools.
---

# Aviation AI assurance screening

Places an aviation AI use case on EASA's proposed AI-level and hazard-class
framework, and writes it up in a way that survives a reviewer.

Needs the Airside Labs aviation tools (`mcp.airsidelabs.com/mcp`).

## The one thing to get right before anything else

**A screen is not a determination.** The `ai_level` and `hazard_class` on a use
case are Airside Labs' assessment — a title-level pass with a confidence, made to
sort a portfolio. A real classification needs a ConOps and a per-application
hazard assessment, and no table can supply either.

Say that plainly wherever you present one. A reader who mistakes a screen for a
finding will make a planning error, and the phrase that prevents it is short:
*"this is a portfolio-level screen, not a classification; a determination needs a
ConOps."*

The framework itself is the **proposed Issue 03** of the EASA Concept Paper
(June 2026, consultation closed August 2026). Numbers and wording may move at
final publication.

## The sequence

**1. Get the screen.** `get_use_case(id)` returns `classification` with the
level, hazard class, a confidence and a one-line rationale, plus a `framework`
block carrying the matching rows with their `cp_ref` page.

**2. Read it against the tables.** `easa_ai_framework()` returns the AI levels,
the hazard classes, the technique ceilings and the source notes. Pass `level` or
`hazard_class` for one row. Use it to explain what 1B/H3 *means* before proposing
a system — most readers will not know, and a level quoted without its definition
is jargon.

**3. Cite the page, not the tool.** Every row carries `cp_ref`. That is the
citation. Quote it.

Be aware the `cp_ref` values are not uniform: the AI-level table sits around
pp. 33-34 with a second treatment around pp. 196-208, and different levels cite
different anchors — 1A gives both, 1B gives only the annex. If you cite several
levels in one document they will look inconsistent. Say which page you read each
from rather than implying one table.

**4. Check the technique ceiling if a technique is named.** The framework caps
which techniques are acceptable to which assurance level. The one that catches
people: **large off-the-shelf models (general-purpose LLMs) ceiling at IDAL D /
AL5 / SWAL4 — H4 uses only** — because the training data and design process are
intractable. If a proposal puts an LLM anywhere near a safety-related function,
that is the sentence to quote.

**5. Put a measured failure mode next to the hazard.** When the record
carries an `edge_case` exhibit, quote it beside the screen: a registration
mark worn by two airframes (N803AL), a Mode-S address heard with its top digit
lost (G-EJCF on two addresses), a zero-padded flight identifier (EVA031), four
airframes under one flight number in thirteen days (BAW49). Each is a concrete
way the AI element's input can be wrong, which is what a hazard argument has
to reason about, and each comes with the exact identifiers for the test plan.
Cite the exhibit with its source and window; it is one receiver or one feed on
one dated edition, not a rate.

## Reading the levels

Pick by what the system *does* and how much authority the end user keeps.

| Level | Does what | User authority |
|---|---|---|
| 0 | Acquires/analyses, no user interaction, no link to decisions | n/a |
| 1A | Supports information acquisition and analysis, **including prediction**; proposes no actions | Full |
| 1B | Supports decisions by **proposing alternative options** | Full |
| 2A | **Takes** directed decisions/actions; user cross-checks and can override every one | Full |
| 2B | Shares tasks under **partial** authority, shared situation awareness | Partial |
| 3A | Decides and acts; human overrides **on alerting only** | Limited |
| 3B | Decides and acts with **no end user** | None |

Most detection, prediction and monitoring systems are **1A**. A recommender that
puts options in front of a person is **1B**. Something that acts and is
cross-checked is **2A**. Do not inflate a monitor into a 2A because it sounds
more advanced — the level is about authority, not sophistication.

## Reading the hazard classes

Hazard is the **worst credible effect if the AI is wrong**, not how important the
system feels.

| Class | Worst credible effect | Assurance at acceptable risk |
|---|---|---|
| H5 | No possible impact on end users or public | AL6 — not a safety component under Reg. (EU) 2024/1689 |
| H4 | Slight performance degradation or slight workload increase | AL5/TQL5 |
| H3 | Significant degradation, potentially impacting safety-critical operations | AL3/TQL3 |
| H2 | Potential serious injury; large degradation | AL2/TQL2 |
| H1 | Potential fatalities | **Not acceptable for AI operation at this time** |

Reason about the actual failure mode, and about what else would have to fail. A
tow-route recommender that could in principle route onto a runway still has ATC
clearance and lighted stop bars between it and a collision — that is H3, not H1.
Ask what barriers remain, and say which ones you are relying on.

Two errors to avoid in equal measure: marking everything H4 out of caution, and
inflating to H1/H2 for drama. Both destroy the screen's usefulness.

## Writing the assurance section

A defensible one is short and states its own limits:

1. **What the system does**, in the level's own vocabulary ("proposes alternative
   options to a controller who retains full authority").
2. **The level and hazard, with the definitions**, not just the codes.
3. **The assurance level implied** at acceptable and moderate risk.
4. **The technique ceiling** if a technique is proposed.
5. **The limits**, in one sentence: portfolio screen not a determination;
   proposed issue, numbers may move; a real classification needs a ConOps.

## Traps

- Presenting a screen as a certification finding or a compliance argument.
- Quoting a level code with no definition.
- Citing the tool rather than the `cp_ref` page.
- Assuming the hazard class follows from the AI level. They are independent axes:
  a 1A monitor can be H2 if its silent failure lets someone be struck.
- Forgetting the proposed-issue caveat, which dates the whole analysis.
