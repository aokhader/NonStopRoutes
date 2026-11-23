# Progress

Update at the end of every session: tick what's done and rewrite **Now / Next / Blocked**. Requirement (R#), walkthrough (#) and test (T#) IDs come from [docs/SPEC.md](../docs/SPEC.md).

## Now / Next / Blocked
- **Now:** Spec and context folder written. **PROPOSED** items ([decisions.md](decisions.md)) are the defaults until you change one, so building doesn't wait on them.
- **Next:** P1: Google Cloud setup checklist, then the skeleton.
- **Blocked:** Google Cloud account with billing (you're setting it up).

## Phases
| Phase | Status | Done when |
|---|---|---|
| P1 Setup | ☐ | A map loads at http://localhost:8000 |
| P2 Logic | ☐ | T1–T25 and T30–T32 pass |
| P3 One leg | ☐ | #1 passes; #4 passes for a 2-stop trip |
| P4 Whole trip | ☐ | #2–9, #12, #16 pass |
| P5 Map | ☐ | #10–11 pass |
| P6 Readable | ☐ | #1, #15 pass; an older user plans a trip unaided |
| P7 Deploy | ☐ | Walkthrough passes on the live URL |
| P8 Extension | ☐ | #13–14 pass; T26–T29 pass |

## Phase briefs

**P1 Setup.** Goal: a key that works and a page that shows a map.
- Files: `.gitignore`, `package.json`, `web/index.html`, `web/style.css` (empty start), `web/config.example.js`, `web/config.js`, `web/maps.js` (`loadGoogle`, `createMap`)
- First tasks: (1) SPEC §6 setup steps 1–6 (you, in the browser); (2) create `.gitignore`, `package.json`, `config.example.js`, then step 7: copy it to `config.js` with your key; (3) `index.html` + `loadGoogle` + `createMap` → a world map shows

**P2 Logic.** Goal: all of `trip.js`, tested, with no Google involved.
- Requirements: R2, R3, R6, R7, R8 (logic), plus `mapsLink` (R12) and `readAddStopMessage` (R14)
- Files: `web/trip.js`, `tests/trip.test.js`
- First tasks: (1) T1–T9 `resolveClock` + `leaveTime`; (2) T10–T13 `planTrip` with a fake `computeLeg`; (3) T14–T25, T30–T32: `summarize`, `toMilli`, formatters, `defaultStart`, `tripDayProblem`, `mapsLink`, `readAddStopMessage`

**P3 One leg.** Goal: two real places → one real part card.
- Requirements: R1 (2 stops), R2, R4, R5
- Files: `web/maps.js` (`attachPlacePicker`, `computeLeg`), `web/app.js`, `web/index.html`
- First tasks: (1) read the current Route + Place Autocomplete reference pages and note exact field names in decisions.md; (2) `attachPlacePicker` on 2 boxes + date/time inputs; (3) `computeLeg` → render one card with steps (ui.md text patterns). Then check the Cloud billing report: does transit bill as Essentials?

**P4 Whole trip.** Goal: 2–8 stops, leave rules, summary, every failure message.
- Requirements: R1, R3, R6, R7, R8, R9
- Files: `web/app.js`, `web/index.html`, `web/style.css`
- First tasks: (1) "+ Add a stop" up to 8 boxes + Leave at / Stay for control (removing stops is R15, P6); (2) wire `planTrip` + progress text; (3) summary card, late warning, no-route and error cards

**P5 Map.** Goal: see the whole trip; jump to Google Maps per part.
- Requirements: R11, R12
- Files: `web/maps.js` (`drawTrip`), `web/app.js`
- First tasks: (1) `createPolylines()` per part; (2) numbered markers (check whether the marker type needs a Map ID); (3) "Open in Google Maps" button using `mapsLink`

**P6 Readable.** Goal: someone's grandparent can use it.
- Requirements: R10, R15, R16
- Files: `web/style.css`, `web/index.html`, `web/app.js`
- First tasks: (1) go through [ui.md](ui.md)'s accessibility list and fix each item; (2) remove / move up / move down; (3) save and restore `nsr:lastTrip`. Then: watch one real older user try it, and fix what they trip on

**P7 Deploy.** Goal: a public link.
- Files: `.github/workflows/pages.yml`
- First tasks: (1) add the `MAPS_API_KEY` secret + Pages source; (2) workflow per [setup.md](setup.md) §5; (3) run the walkthrough on the live URL

**P8 Extension.** Goal: the app in Chrome's side panel, plus one-click add from Google Maps.
- Requirements: R13, R14
- Files: `extension/*`, `tests/mapsurl.test.js`, `web/app.js` (message listener)
- First tasks: (1) T26–T29 + `parseMapsUrl`; (2) manifest + background + side panel iframe, and check framing works (if not, take the fallback in SPEC §9); (3) Add this place → `postMessage` → app adds the stop

## Requirements
| ID | Priority | Phase | Status |
|---|---|---|---|
| R1 | Must | P3, P4 | ☐ |
| R2 | Must | P2, P3 | ☐ |
| R3 | Must | P2, P4 | ☐ |
| R4 | Must | P3 | ☐ |
| R5 | Must | P3 | ☐ |
| R6 | Must | P2, P4 | ☐ |
| R7 | Must | P2, P4 | ☐ |
| R8 | Must | P2, P4 | ☐ |
| R9 | Must | P4 | ☐ |
| R10 | Must | P6 | ☐ |
| R11 | Must | P5 | ☐ |
| R12 | Should | P2, P5 | ☐ |
| R13 | Must | P8 | ☐ |
| R14 | Must | P2, P8 | ☐ |
| R15 | Should | P6 | ☐ |
| R16 | Should | P6 | ☐ |
| R17 | Could | later | ☐ |
| R18 | Could | later | ☐ |

## Walkthrough rows
| # | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Pass | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
