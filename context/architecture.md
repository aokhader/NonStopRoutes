# Architecture

How the pieces fit. Data shapes and function signatures are in [contracts.md](contracts.md); the why is in [../docs/SPEC.md](../docs/SPEC.md) §5.

## Repo layout
| Path | What it is | Phase |
|---|---|---|
| `web/index.html` | The one screen | P1 |
| `web/style.css` | All styles (rules in [ui.md](ui.md)) | P1, P6 |
| `web/app.js` | Entry point: reads the form, keeps state, renders results. No exports | P3–P6 |
| `web/trip.js` | **Pure logic:** leave times, the leg chain, totals, formatting, message checks | P2 |
| `web/maps.js` | **Only file that talks to Google:** loadGoogle, createMap, place picker, computeLeg, drawTrip | P1, P3, P5 |
| `web/config.example.js` | Template for the API key (committed) | P1 |
| `web/config.js` | Real API key (**gitignored**) | P1 |
| `tests/trip.test.js` | T1–T25 | P2 |
| `tests/mapsurl.test.js` | T26–T29 | P8 |
| `extension/manifest.json` | MV3 manifest: `sidePanel` permission, host permission `https://www.google.com/maps/*` | P8 |
| `extension/background.js` | Makes the toolbar icon open the side panel | P8 |
| `extension/sidepanel.html` + `sidepanel.js` | iframe of the web app + "Add this place" button | P8 |
| `extension/mapsurl.js` | Pure: Google Maps place URL → `Place` | P8 |
| `.github/workflows/pages.yml` | Deploy `web/` to GitHub Pages, writing `config.js` from a secret | P7 |
| `package.json` | `{"type":"module","private":true,"scripts":{"test":"node --test"}}`, no dependencies | P1 |

## Process model
- **No server of ours.** The web app is static files. The browser talks to Google directly with a website-restricted key.
- **Extension:** a Chrome side panel page (local to the extension) that holds the hosted web app in an iframe. The extension never calls Google itself.

## Path of one "Plan my trip" click
1. `app.js` reads the form → builds a `Trip` (start `Date` + stops). It validates first: 2–8 stops, each stop picked from suggestions or added by the extension, stay 1–720 min, and `tripDayProblem(start, now)` returns `null`.
2. `app.js` calls `planTrip(trip, computeLeg, onProgress)` from `trip.js`.
3. For leg *i* (stop *i* → stop *i+1*), in order:
   1. `readyAt` = trip start (leg 1) or `leaveTime(rule, arriveAt).leaveAt` of stop *i* (rule in SPEC §7).
   2. `onProgress(i, n)` → app shows "Planning part *i* of *n*…".
   3. `computeLeg(from, to, readyAt)` in `maps.js` → **crosses the trust boundary:** one `Route.computeRoutes` request to Google, `travelMode: 'TRANSIT'`, no intermediates.
   4. Google's route → `maps.js` converts it to a plain `LegData` (steps, times, fare as `{currency, milli}`), or `null` if there's no route.
   5. `null` → leg status `noRoute`, **stop the chain**. Thrown error → status `error`, stop the chain.
4. `summarize(trip, legs)` → end time, total minutes, fare totals per currency, legs missing a fare.
5. `app.js` renders summary + part cards, then `drawTrip(map, legs)` draws on the map made once at page load by `createMap(mapEl)`.

Legs run **one after another, not in parallel**. Each leg's departure depends on the previous leg's arrival.

## Path of "Add this place" (extension)
1. The user clicks **Add this place** in the side panel.
2. `sidepanel.js` → `chrome.tabs.query({active: true, currentWindow: true})` → `tab.url`. This needs the host permission, otherwise `url` is missing.
3. `parseMapsUrl(tab.url)` → `Place`, or `null` → show "Open a place in Google Maps first."
4. `iframe.contentWindow.postMessage({type: 'nsr:add-stop', name, lat, lng}, APP_ORIGIN)`.
5. In the app, `app.js` only accepts the message if `event.source === window.parent`, `event.origin` starts with `chrome-extension://`, and `readAddStopMessage(event.data)` isn't `null`. It then appends the stop before the final stop's empty box (or as the next stop).

## Screen states (app.js)
`empty` → (user types) → `editing` → (Plan my trip) → `planning` → `results` | `partial` (no route / leg error) | `failed` (Google didn't load)
- Any edit in `results`/`partial` → back to `editing`, and old results are cleared (walkthrough #12).
- `failed` → "Try again" reloads Google.

## Failure table
| Dependency | What goes wrong | What the user sees | Fallback |
|---|---|---|---|
| Maps JS API load | Offline, bad key, quota | "Can't reach Google Maps. Check your internet, then press Try again." | Try again button |
| Place suggestions | Load fails | Same banner | — |
| `computeLeg` | Network/quota error, or no answer in 15 s (**PROPOSED** timeout) | Leg card: "Couldn't plan this part. Check your internet and press Plan my trip again." | Chain stops; earlier legs stay |
| `computeLeg` | Empty routes | "No bus or train found for this part at this time. Try another time or stop." | Chain stops |
| Fare | Missing for a leg | "Fare not available" on that leg; summary says "At least …" | — |
| Date out of range | >100 days ahead | "Pick a day within the next 100 days." | Blocked before any request |
| Map drawing | Fails | The list still shows; map area says "Map not available" | — |
| Extension iframe | App offline / framing blocked | Side panel: "Can't load NonStopRoutes. Check your internet." | Button opens the app in a new tab |
| `parseMapsUrl` | Not a place page / URL format changed | "Open a place in Google Maps first." | Fix parser against T26–T29 |

## Speed budget (PROPOSED)
- Google load + first map: ≤3 s on 4G.
- Each leg ≈1 s → an 8-stop trip (7 legs) ≤10 s. Progress shows per leg, so the wait is visible.

## Words used in these docs
| Word | Meaning |
|---|---|
| Leg (screen: "part") | The trip between two stops next to each other in the list |
| Pure function | Uses only its inputs, changes nothing outside itself, gives the same answer every time. Easy to test |
| Trust boundary | The line where data leaves our code: here, only `maps.js` → Google, and extension → app |
| Opaque | A value other files pass along but never look inside (`googleRoute`) |
| int64 / nanos | How Google sends money: whole units (sometimes as text) plus billionths ("nanos") |
| Live region | A part of the page a screen reader announces when it changes (`aria-live`) |
| Map ID | A Google Cloud setting some map features need; check in P5 whether ours do |

## Cost per click
One "Plan my trip" = (stops − 1) Compute Routes requests. Plus one place-suggestion session per picked stop and one map load per page load. Free caps in SPEC §6.
