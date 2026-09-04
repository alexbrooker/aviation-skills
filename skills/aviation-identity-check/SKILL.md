---
name: aviation-identity-check
description: Resolve and cross-check aviation identifiers — airport codes, airline designators, aircraft type designators, registrations, flight numbers — with temporal correctness and provenance. Use when identifying what a code refers to, especially on a past date, when reconciling identifiers from different systems, or when validating a set of identifiers against each other before trusting a record. Uses the Airside Labs aviation MCP tools.
---

# Aviation identity resolution

Resolves aviation identifiers to canonical entities with a confidence, the
alternates, temporal validity and per-field provenance — and cross-checks a set
of them against each other.

Needs the Airside Labs aviation tools (`mcp.airsidelabs.com/mcp`).

## The rule that matters most

**Pass `as_of` for anything historical.** Codes are reused, and an undated answer
is silently wrong for past data.

- `HKG` meant Kai Tak until 1998 and Chek Lap Kok after it.
- The designator `SN` was Sabena's; Brussels Airlines carries it now. Ask about
  1995 without a date and you may be offered the later holder as an alternate
  for a year in which it did not yet exist.
- Airport codes move between aerodromes when a city replaces its airport, which
  has happened in more places than most people expect.

Without a date you get today's answer, applied to a record from 1999, and nothing
in the response will look wrong. This is the single most common way to produce a
confident error with these tools.

## The tools

| Tool | Answers |
|---|---|
| `resolve_airport` | IATA/ICAO code, name fragment or city → aerodrome, with location, elevation, IANA timezone, type |
| `resolve_airline` | designator or name → airline, with country and validity window |
| `resolve_aircraft_type` | type designator or model name → ICAO type, manufacturer, model |
| `resolve_registration` | registration mark → airframe, with Mode S address, type, operator, validity |
| `parse_flight_identifier` | flight designator → carrier and number, with the parse rule applied |
| `validate_identifiers` | a SET of identifiers → whether they are consistent with each other on a date |

## Reading a result

`status` is `resolved`, `ambiguous` or `unresolved`. **An unresolved answer is a
real result, not an error** — it means the dataset cannot identify the thing, and
inventing one would be worse. Report it as an answer.

`confidence` is about how certain the *match* is, not how good the source is:

- **1.0** — exact unique match on an unambiguous identifier for the `as_of` date.
- **0.7–0.99** — unique match via normalisation, an alias, or a historical record.
- **0.4–0.69** — best of several plausible candidates, and `alternates` is
  populated. Prefer asking the user over picking one.
- **below 0.4** — speculative. Do not act on it.

Source quality lives in `provenance`, per field, where it can be inspected —
deliberately not folded into the confidence number.

`corroboration` says how many independent sources stand behind the record:
`corroborated` (two or more agree), `asserted` (one official register — an
authority, not a guess), `bootstrap` (one aggregator's claim), `observed` (only
a direct observation stands behind it). A caller deciding whether to act needs
this more than another decimal place of confidence.

An ambiguous name returns candidates rather than picking one. Narrow it with
`country_hint` (ISO 3166 alpha-2) rather than guessing.

## Cross-checking a set

`validate_identifiers` is the one to reach for when a record arrives from a
system you do not control — a schedule row, a movement message, a spreadsheet —
and you want to know whether its parts agree before trusting it.

Read the verdict precisely, because the vocabulary is doing real work:

- `consistent` — every decisive check ran **and** agreed.
- **`consistent_where_checkable`** — the checks that ran agreed, but some were
  `unverifiable`. This is not the same claim, and the difference matters: the
  silent checks are often exactly the ones that would have caught a wrong type
  or operator, typically because no airframe record exists for that registration.
  Treat it as "nothing contradicted", never as "verified".
- `unverifiable` on a single check means the dataset cannot say — a real answer,
  not a failure.

Look at the per-check detail and the unverifiable count, not just the headline.

A contradiction is a finding worth reporting plainly: it usually means one of the
identifiers is wrong, not that the tool is.

## Traps

- **Omitting `as_of` on a historical question.** The one that produces confident
  errors most often.
- Reading `unresolved` as a failure and retrying with a made-up variant.
- Acting on a sub-0.4 confidence.
- Taking a `consistent` verdict without checking how many checks were
  unverifiable.
- Using these for live operational status — whether an airport is accepting
  traffic, slots, runway state. They answer identity questions only.
- Assuming a Mode S address implies a country. Deriving one from the address is
  deliberately not offered, because the allocation table is not ours to reproduce.
- Treating a registration as a permanent name. It is a window: marks are
  withdrawn and re-issued, and the same mark can belong to different airframes at
  different times. That is why the validity dates are there.
