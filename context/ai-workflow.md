# AI workflow

Rules for every Claude Code session on this repo.

## Start
1. Read [CLAUDE.md](../CLAUDE.md), then [progress.md](progress.md): the current phase's brief and the Now / Next / Blocked section.
2. Read [contracts.md](contracts.md) for every function you'll write or call.
3. Say the **one task** you'll do this session and which requirement IDs (R#) and test IDs (T#) it covers.

## While working
- **Stay inside the current phase.** If the task needs another phase's work or a contract change, stop and say so before doing it.
- **Plan in 3–5 lines, then build the smallest version that works.** Run it and show the output (test results, or what the page shows).
- **Check docs before any Google or Chrome API call.** Use the current reference pages linked in SPEC §6/Sources, not memory. Field names in the Route class have changed between versions.
- **Tests first for `trip.js` and `mapsurl.js`:** write the T# test, see it fail, make it pass.
- **Never invent data.** No sample fares or times in the app. Tests use clearly fake data.
- **Never print, log or commit the API key.**
- **Stuck for 20 minutes?** Take the fallback in [architecture.md](architecture.md)'s failure table or SPEC §9, and log it in [decisions.md](decisions.md).

## Ask first
- Adding any dependency, CDN script or new file type.
- Deleting or renaming files.
- Any destructive git command (reset, force-push, branch delete).
- Changing a contract, a hard rule in CLAUDE.md, or a **PROPOSED** item.
- Committing or pushing.

## End
1. Run `npm test`. Say what passed and what failed. Don't claim a walkthrough row passed unless it was actually checked in a browser.
2. Update [progress.md](progress.md): tick requirements/rows and update Now / Next / Blocked.
3. Add any design choice to [decisions.md](decisions.md).
4. Suggest a commit message in the `[P#] verb: what (R#)` form.
5. Suggest `/clear` before the next task so the next session starts clean.
