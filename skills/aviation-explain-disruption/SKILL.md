---
name: aviation-explain-disruption
description: Explain after the fact why a specific US flight was late, cancelled or diverted, using what the flight actually did, where its aircraft had been that day, what was restricted at the airport and how days like it went before. Use for a passenger, a manager or a post-event review, or to put dated precedent beside a decision about a bad day. Historic records only — not live, not a forecast, not for operational use. Uses the Airside Labs aviation MCP tools.
---

# Explain a disrupted flight

An explanation here is assembled from four dated records, each saying one
thing: what the flight did, what its aircraft had been doing, what was
restricted at the airport, and how similar days ended. None of them is live.
The skill is the order to ask in, and the claims the records do not support.

Needs the Airside Labs aviation tools (`mcp.airsidelabs.com/mcp`).

## What each source can say

| Tool | Record | Covers | It can say | It cannot say |
|---|---|---|---|---|
| `flight_departure_lookup` | Airside Labs' capture of the FAA flight feed | Departures from JFK and Newark from 13 Aug 2026, Seattle from 16 Sep 2026 | When the flight took off (UTC), on which registration and type, to where | Anything about other departure airports |
| `disruption_precedents` | US DOT on-time record | US domestic flights of the large reporting carriers, July 2024 to two or three months ago | Minutes late, the carrier's own cause attribution, the aircraft's other legs, how bad days ended | An airport's total traffic; why a diversion happened |
| `notam_search` | Airside Labs' capture of the FAA NOTAM feed | Daily from 12 Jul 2026, full volume from 30 Sep 2026 | Which notices were in force at a place and time | That a notice caused anything |

The windows overlap only partly. Say which record an answer comes from, and
say when one of them does not reach the date asked about.

## The sequence

**1. Get the identifier into both forms.**

    parse_flight_identifier("UA2815")     -> iata_form UA2815, icao_form UAL2815

The flight feed uses the ICAO form; the on-time record uses the two-character
form.

**2. Find what the flight did.** The feed files a flight under the UTC date of
its take-off.

    flight_departure_lookup("UAL2815", "<date>", "KEWR")

If it answers `none_found`, read `plans_without_wheels_up` and
`nearest_dates_with_this_callsign`, then ask again for the next UTC date. A
flight delayed past midnight UTC is under the next day. `not_captured` means
the airport or date is outside the capture: it is never evidence that nothing
flew.

**3. Follow the aircraft, not the flight number.** Here the date is the LOCAL
scheduled date.

    disruption_precedents(kind="aircraft_day", flight="UA2815", like_date="<local date>")

Read the flight's own leg: `departure_minutes_late`,
`carrier_attributed_minutes`, and `inbound_arrived_after_scheduled_departure_min`
(above zero, the aircraft landed after this flight was due to leave). Check the
`registration` is the one step 2 gave.

**4. Read what was restricted at the airport at the scheduled time.**

    notam_search("KEWR", at="<scheduled departure, UTC>")
    notam_coverage("KEWR")                -> how complete the capture was that day

**5. Ask whether the day was unusual, and how days like it went.**

    disruption_precedents(kind="airport_days", airport="EWR", like_date="<local date>")

`the_day` is the airport's day; `precedents` are its other disruptions, matched
on the share cancelled and the coded cause; `annotation.pattern` is a count.
For a diverted flight, add
`disruption_precedents(kind="diversions", airport="<destination>")`.

**6. Write it in this order:** what happened to the flight; what the carrier
attributed it to; what the aircraft had been doing; what the airport's day was
like; how similar days ended; the sources with their windows.

## Traps

- **Two clocks.** The on-time record is local time and local date. The flight
  feed and NOTAMs are UTC. Convert before you compare, and say which you quote.
- **Same flight number, different flights.** A delayed flight and the next
  day's flight can share a UTC date. Match on the registration.
- **A NOTAM is context, never the cause.** Say a runway was closed. Do not say
  it delayed the flight.
- **The cause is the carrier's.** "Weather" means the carrier coded those
  minutes as weather. Nobody independent assigned it.
- **A diversion has no reason in these records.** You can say where a flight
  landed and how late it arrived, never why.
- **Reporting carriers only.** A count from the on-time record is never the
  airport's total.
- **A pattern is a count.** "14 of 21 cleared within two days" is a fact about
  21 days at one airport in one window. Quote it with the window.
- **Check coverage before trusting an empty answer.** A thin day or a date
  before a capture began is not "nothing happened".
- **Not for operational use.** The FAA captures are historic, redistributed by
  Airside Labs and not endorsed by the FAA; the on-time record is not endorsed
  by BTS or DOT. Say so when you quote them.

## Worked example — United 2815, Newark to Houston, 27 August 2026

Scheduled 15:59 local. It left the gate at 00:07 the next morning.

```
parse_flight_identifier("UA2815")
   → UA2815 / UAL2815

flight_departure_lookup("UAL2815", "2026-08-27", "KEWR")
   → none_found; a plan seen 19:41Z to 02:12Z that never got airborne;
     nearest dates include the 28th

flight_departure_lookup("UAL2815", "2026-08-28", "KEWR")
   → wheels up 04:43Z, N27511, B39M
     (a second UAL2815 that UTC day is N17583: the next day's flight)

disruption_precedents(kind="aircraft_day", flight="UA2815", like_date="2026-08-27")
   → N27511, three legs. EWR–MIA on time; MIA–EWR arrived 16:02 against
     14:30; UA 2815 left 488 minutes late, the inbound having landed 3
     minutes after its scheduled departure. Carrier attribution: weather
     425, late aircraft 58.

notam_search("KEWR", at="2026-08-27T20:00:00Z")
   → 14 in force, among them RWY 11/29 CLSD, 11:07Z to 21:00Z.
     Capture warning: about half volume before 30 Sep 2026.

disruption_precedents(kind="airport_days", airport="EWR", like_date="2026-08-27")
   → 109 of 334 reporting-carrier departures cancelled: 61 weather, 38 air
     traffic system, 10 carrier. Next day 4.8%. Of 21 weather-coded
     disruptions at Newark in the window, 14 cleared within two days.
```

Two records agree on the take-off without knowing about each other: the
on-time record says wheels off at 00:43 local, the flight feed says 04:43Z,
which is 00:43 in Newark. Both name N27511.

The explanation:

> United 2815 from Newark to Houston on 27 August 2026 left 488 minutes late,
> taking off at 00:43 the next morning. United attributed 425 of those minutes
> to weather and 58 to the aircraft arriving late: the same aircraft, N27511,
> had come in from Miami 92 minutes behind schedule, landing after UA2815 was
> due to leave. It was a bad day at Newark. A third of reporting-carrier
> departures were cancelled, most coded as weather or air traffic system, and
> runway 11/29 was closed by NOTAM from 11:07Z to 21:00Z. Newark was back under
> 5% cancelled the next day; of 21 weather-coded disruptions there between
> July 2024 and August 2026, 14 cleared within two days.
> Sources: US DOT BTS on-time performance, edition 2026-08 (reporting carriers
> only; causes are the carrier's own); Airside Labs' historic capture of the
> FAA flight and NOTAM feeds (not for operational use, not endorsed by the FAA;
> NOTAM capture partial before 30 September 2026).

It does not say the runway closure caused the delay. Nothing here supports that.
