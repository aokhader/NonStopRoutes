# Decisions

Newest at the bottom. Before changing anything here, add a new row saying what changed and why. Don't edit old rows.

## Log
| Date | Decision | Why |
|---|---|---|
| 2026-11-22 | Web app + Chrome extension, one codebase | Google Maps has no third-party plugin system; phones need a web app, desktop users asked for the extension |
| 2026-11-22 | Extension = side panel showing the hosted app + "Add this place" from google.com/maps | Reuses the app; one-click add is the reason to have an extension at all |
| 2026-11-22 | Each middle stop: "Leave at" clock time **or** "Stay for" minutes | User's choice; both are natural ways to think about errands |
| 2026-11-22 | Works anywhere Google has transit; fare only where Google has it | Google returns a fare only when it knows every step's fare |
| 2026-11-22 | Solo, no deadline → build phases P1–P8 instead of a clock schedule | Project setting |
| 2026-11-22 | Plain HTML/CSS/JS ES modules, no framework or build step | Easiest to read; one screen doesn't need more |
| 2026-11-22 | Spec in `docs/SPEC.md` | Lives with the code |
| 2026-11-22 | One `Route.computeRoutes` request per leg, run in order | Transit rejects intermediate waypoints; each leg depends on the last arrival |
| 2026-11-22 | Maps JS `Route` class + `PlaceAutocompleteElement`, not legacy DirectionsService/Autocomplete | Legacy services are being replaced |
| 2026-11-22 | No backend; browser key with website + API restrictions, quotas, budget alert | Browser keys are public by design; restrictions are the protection |
| 2026-11-22 | Key in gitignored `web/config.js`; GitHub Actions writes it on deploy | Keeps the key out of git history |
| 2026-11-22 | Hosting on GitHub Pages | Free, same repo |
| 2026-11-22 | "Leave at" time = that clock time nearest the arrival (within 12 h) | Late arrivals and too-early times become visible warnings; still handles midnight. shortcut: no stays over 12 h |
| 2026-11-22 | PROPOSED items are the defaults until changed | Building doesn't wait on sign-off; each is a small edit later |
| 2026-11-22 | Fares stored as integer thousandths per currency | Exact sums for 0-, 2- and 3-decimal currencies |
| 2026-11-22 | Times shown in the browser's time zone | shortcut: fine when planning locally, upgrade if users plan trips in other cities |
| 2026-11-22 | A no-route or failed leg stops the chain | Later legs would depend on an unknown arrival time |

## Needs your sign-off (PROPOSED)
| Item | Proposal | Where |
|---|---|---|
| Max stops | 8 | SPEC R1 |
| Default for new middle stops | Stay for 30 min | ui.md |
| On-screen word for a leg | "Part" ("Part 1: Home → Pharmacy") | ui.md |
| Body text size | 20px | ui.md |
| Colors | Table in ui.md | ui.md |
| Google call timeout | 15 s | architecture.md |
| Speed budget | ≤3 s first map, ≤10 s for 8 stops | architecture.md |
| Daily quota caps | Routes 500, Places 1,000, Maps loads 500 | SPEC §6 step 5 |
| Language | English only for now | SPEC §2 |

## To confirm while building
| Question | Phase |
|---|---|
| Does a transit request bill as Essentials? (check the billing report) | P3 |
| Exact Route response field names for steps, transit details and fare | P3 |
| Which Cloud APIs the `Route` class needs (assumed: Routes API) | P1/P3 |
| Can the side panel iframe the GitHub Pages app? | P8 |
| Does the marker type need a Map ID? | P5 |
