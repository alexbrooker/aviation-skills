# Airside Labs aviation skills

Agent skills for the [Airside Labs aviation MCP tools](https://airsidelabs.com/tools)
— aviation identifier resolution, an AI use-case catalogue, EASA assurance
screening and airport facts, at `mcp.airsidelabs.com/mcp`.

```
npx skills add alexbrooker/aviation-skills
```

| Skill | For |
|---|---|
| `aviation-opportunity-map` | Building an AI opportunity map, briefing deck or PRD for an aviation organisation |
| `aviation-ai-assurance` | Screening a use case against EASA's proposed AI-level and hazard framework |
| `aviation-identity-check` | Resolving and cross-checking aviation identifiers, with temporal correctness |
| `aviation-data-story` | Grounding a PRD, data requirements section or test plan in measured findings (exhibits) served with the catalogue |

## What these are

**Method, not data.** These skills teach an agent how to ask: which tool to reach
for, in what order, what the vocabulary means, and — mostly — the traps that
produce a confident wrong answer. The data stays behind the tools.

Exhibits are the one place the data comes forward: a use-case record may carry
up to three measured, dated findings from Airside Labs' own ADS-B receiver, the
FAA flight-plan feed, BTS traffic statistics or the served registry, each with
the exact identifiers to test with and the scope it must not be read past.
They are static and verified, never live; the `aviation-data-story` skill says
how to cite them.

That is deliberate. The most valuable thing in this repository is the list of
ways to get an aviation question wrong: searching capability labels instead of
operational nouns, counting raw requirement titles as distinct feeds, omitting
`as_of` on a historical identifier, or reading a portfolio screen as a
certification finding. An agent that knows those four things gets better answers
on its first attempt than one that discovers them over twenty calls.

## Trying it without an account

`use_case_landscape`, `get_use_case`, `submit_suggestion` and `feedback_status`
need **no credential at all** (a per-address daily cap applies). You can scope a
catalogue and pull full records before deciding whether any of this is useful.

## Feedback

`submit_suggestion` is keyless and is the intended channel: corrections, missing
use cases, data sources we should know about, or a tool that would have helped.
`report_unmet_need` is for the specific case where you looked for something and
it was not there — an empty result is a real answer, and telling us is how the
gap gets closed.

## Contributing

Corrections are welcome, particularly to the traps: if one of these skills led
you to a wrong answer, that is the most useful thing you can tell us. Open an
issue, or use `submit_suggestion` from the tools themselves.

## Licence

The skills in this repository are MIT. The data they reach is not: it is served
under the Airside Labs subscription terms, and each tool response carries a
`provenance` block saying what you may do with it.
