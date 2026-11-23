# NonStopRoutes

Plan a bus/train trip with several stops at once. For each stop the user says "Leave at 2:30 PM" or "Stay 45 minutes". The app shows every leg with times, when the trip ends, and the total fare. It's a plain web app plus a Chrome extension side panel, built for users of all ages.

Full spec: [docs/SPEC.md](docs/SPEC.md). Requirement IDs (R1–R18), walkthrough rows (#1–16) and test IDs (T1–T32) come from it. Use them in commits and notes.

## Hard rules
1. **Plain HTML/CSS/JavaScript ES modules.** No framework, no build step, no npm dependencies unless the user says yes.
2. **`web/maps.js` is the only file that calls Google.** Everything else gets plain data from it.
3. **`web/trip.js` is pure.** No DOM, no Google, no `Date.now()` inside: pass times in. Every function there has a test.
4. **One transit request per leg.** Google rejects intermediate stops for transit, so never send `intermediates`.
5. **Never invent data.** No made-up fares, times or places. Missing fare → "Fare not available".
6. **No secrets in git.** The API key lives only in `web/config.js` (gitignored). Never print it.
7. **Readable for all ages:** text ≥18px, tap targets ≥48px, plain words, never color alone (see [context/ui.md](context/ui.md)).
8. **Simple beats clever.** Smallest code that fully works. Comment only the *why*.
9. **Check Google/Chrome docs before writing an API call.** Field names change, so don't code from memory.

## Phases (status in [context/progress.md](context/progress.md))
| Phase | Goal | Requirements |
|---|---|---|
| P1 | Cloud setup + map shows on localhost | — |
| P2 | `trip.js` + tests T1–T25, T30–T32 | R2, R3, R6–R8 logic; helpers for R12, R14 |
| P3 | One leg end-to-end | R1, R2, R4, R5 |
| P4 | Whole trip: chain, summary, fares, warnings | R1, R3, R6–R9 |
| P5 | Map + Open in Google Maps | R11, R12 |
| P6 | Readability pass, edit stops, remember trip | R10, R15, R16 |
| P7 | Deploy to GitHub Pages | — |
| P8 | Chrome extension | R13, R14 |

## Where to look
| File | Read it when |
|---|---|
| [context/progress.md](context/progress.md) | **Every session start.** Current phase, its brief, what's next |
| [context/architecture.md](context/architecture.md) | Touching how files connect, the leg chain, or errors |
| [context/contracts.md](context/contracts.md) | Writing or calling any exported function or message |
| [context/ui.md](context/ui.md) | Any HTML, CSS or wording |
| [context/code-standards.md](context/code-standards.md) | Before writing code or committing |
| [context/ai-workflow.md](context/ai-workflow.md) | How a session runs, step by step |
| [context/setup.md](context/setup.md) | Google Cloud, API key, running, deploying, loading the extension |
| [context/decisions.md](context/decisions.md) | Before changing a design choice; log new ones here |
| [docs/SPEC.md](docs/SPEC.md) | Requirements, walkthrough rows, test cases, risks |

## Commands
```
python -m http.server 8000 -d web     # run the app → http://localhost:8000
npm test                              # node --test, all tests
```

## Session steps
**Start:** read this file, then `context/progress.md` (your phase's brief), then `context/contracts.md` for any function you touch. Name the one task you'll do.
**End:** run `npm test`, update `context/progress.md` (Done / Next / Blocked), add any design choice to `context/decisions.md`, then suggest a commit message `[P#] verb: what (R#)`.
