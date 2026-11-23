# Code standards

Goal: code someone new to programming can read top to bottom.

## JavaScript
- Plain ES modules (`import`/`export`), modern syntax (Node 22 and current Chrome both support it). No TypeScript, no framework, no bundler.
- Named exports only. `const` by default, `let` when reassigned, never `var`.
- Small functions with plain names (`leaveTime`, not `computeEffectiveDepartureTs`).
- JSDoc on every exported function, using the typedef names in [contracts.md](contracts.md).
- 2-space indent, single quotes, semicolons.
- No new npm dependency, CDN script, or library without the user's yes. Google's Maps JS API is the only outside script.

## Comments
- Comment only the *why* the code can't show, in one line.
- Known limits use this exact form: `// shortcut: <the limit>, <when to upgrade>`
  - e.g. `// shortcut: times shown in the browser's time zone, upgrade if users plan trips in other cities`

## Errors and network
- `maps.js` throws `Error` with a plain message; `app.js` catches it and shows the matching text from [ui.md](ui.md).
- Never swallow an error silently: show it, or `console.error` it *and* show a message.
- Every Google call has a timeout (15 s, **PROPOSED**) via `Promise.race`.
- Validate at the edges: form input (stop count, stay minutes, date range) before planning; extension messages with `readAddStopMessage`; saved trips on load.

## Secrets
- The API key lives only in `web/config.js` (gitignored); `web/config.example.js` holds a placeholder.
- Never `console.log` the key or a full request URL that contains it.
- `.gitignore` must contain: `web/config.js`, `.env`, `node_modules/`.

## Dates and times
- Pass "now" in as an argument; `trip.js` never calls `Date.now()` or `new Date()` with no arguments.
- Tests build dates with local parts: `new Date(2026, 10, 22, 9, 0)`. **The month is 0-based (10 = November).**
- **Never** `new Date('2026-11-22')`: a date-only string is read as UTC midnight, which can be the previous day locally. Build dates from parts, or parse `"2026-11-22"` + `"09:00"` yourself.
- Avoid daylight-saving change dates in tests.

## Tests
- `npm test` runs `node --test`; files live in `tests/` and end in `.test.js`.
- Test every function in `web/trip.js` and `extension/mapsurl.js` using the T-IDs in [SPEC §7](../docs/SPEC.md#7-logic-formulas-and-test-cases). Put the ID in the test name: `test('T7 late arrival warns', …)`.
- `maps.js` and `app.js` are checked by the walkthrough rows in SPEC §3, not unit tests.
- A phase isn't done until `npm test` passes.

## Git
- `main` always runs. Work on a branch per phase (`p3-one-leg`) and merge when its walkthrough rows pass.
- Commit message: `[P3] add: computeLeg for one transit leg (R4)`.
- Never commit `web/config.js`, `.env`, keys, or screenshots containing a key. Never force-push `main`.
- Commit only when the user asks.

## Windows notes
- Use `python`, not `python3`. Run commands from the repo root.
- The local server must be on port **8000**, because the API key only allows `http://localhost:8000/*`.
