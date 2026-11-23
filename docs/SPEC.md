# NonStopRoutes: Technical Spec

Status: draft for sign-off · 2026-11-22 · Owner: Abdulaziz Khader (solo)

Tags used below: **PROPOSED** = my choice, waiting on your OK (also in [context/decisions.md](../context/decisions.md)). **ASSUMPTION** = not confirmed in docs yet; check it in the phase named.

---

## 1. Overview

NonStopRoutes plans a public-transit trip with several stops in one go. You list your stops, and for each one say either when you'll leave it ("Leave at 2:30 PM") or how long you'll stay ("Stay 45 minutes"). It then shows the whole trip: every bus and train with times, when the trip ends, and the total fare.

**The problem.** Google transit directions take one destination at a time. Google's routing service rejects transit requests that include stops in between ([Routes API: transit route](https://developers.google.com/maps/documentation/routes/transit-route); [compute routes reference](https://developers.google.com/maps/ai/maps-experience-agent-tools/compute-routes): such a request returns `INVALID_ARGUMENT`). Driving and walking directions allow extra stops. Someone running errands by bus has to re-plan at every stop and can't see ahead of time whether the whole day works.

**The fix.** Split the trip into legs (stop 1→2, 2→3, …). Ask Google for each leg separately, with each leg's own departure time, then put the results back together into one trip.

| Who | What they do | Where |
|---|---|---|
| Commuter, any age | Plans a day of errands by bus/train | Web app on a phone or computer |
| Chrome user | Finds places on google.com/maps, adds them as stops with one click | Chrome extension side panel |
| Helper (family member, carer) | Plans a trip for someone else and passes it on | Web app (share/print are Could items) |

## 2. Goals and non-goals

**Goals.** At the end of Phase 8:
1. A user plans a trip with 2–8 stops in under 2 minutes, without reading instructions.
2. Every stop takes either "Leave at" or "Stay for".
3. The result shows each leg's buses/trains with times, the trip's end time, its total length, and the total fare (or says plainly where a fare isn't known).
4. Text is large and readable by older users, and the app works on phones and desktops.
5. The same app runs in Chrome's side panel and can add the place open on google.com/maps.

**Non-goals.** Not planned:
- Accounts, logins, or storing trips on a server.
- Turn-by-turn navigation or live vehicle tracking. "Open in Google Maps" covers this.
- Buying tickets, or fare rules such as transfer discounts and daily caps. The total is the sum of Google's per-leg fares.
- Reordering stops for the shortest trip. The user's order is kept.
- Driving, cycling, or ride-share legs. Walking only appears inside a transit leg.
- Languages other than English (**ASSUMPTION**, see Open questions).
- A backend server.

## 3. Acceptance walkthrough

The app is done when every row passes. Use 3–4 real places in a city with transit: A (start), B, C, and D, where D has no transit nearby (e.g. a remote rural address).

| # | Step | What happens | Passes when |
|---|---|---|---|
| 1 | Open the app on a phone | One screen: date (today), start time (now, rounded up to 5 min), 2 empty stop boxes, a big **Plan my trip** button | Fits without sideways scrolling; text ≥18px |
| 2 | Start time 9:00 AM. Type A, B, C (add a box for C). B: stay 45 min | Suggestions appear while typing; picking one fills the box | All 3 stops show their picked names |
| 3 | Press **Plan my trip** | "Planning part 1 of 2…" then results (the screen says "part" for a leg) | Results show in ≤10 s; screen reader announces progress |
| 4 | Read leg 1 | Steps: walk 5 min → Bus 12 toward Downtown 9:07 AM, 6 stops, get off at Main St → walk 3 min. Arrive B 9:31 AM | Every transit step shows line, direction, board/exit stop, times, stop count |
| 5 | Read leg 2 | Leaves B at arrival + 45 min | Leave time = leg 1 arrival + 45 min exactly |
| 6 | Read summary (top) | "Trip ends 11:02 AM · 2 h 2 min" and "Total fare $5.50" | End time = last leg's arrival; fare = sum of legs |
| 7 | **Failure: late.** Set B to "Leave at" a time before you arrive | Warning on B: "⚠ Warning: You'll arrive at 9:31 AM, after your 9:20 AM leave time. Part 2 is planned from 9:31 AM." | Warning shown with an icon and words, not just color |
| 8 | **Failure: no route.** Add D after C | Part C→D: "No bus or train found for this part at this time. Try another time or stop." | Legs before D still show; nothing after D is planned |
| 9 | **Failure: no fare.** A leg where Google has no fare | That leg: "Fare not available". Summary: "At least $2.75 (fare unknown for part 2)" | Never shows a made-up or $0 fare |
| 10 | Map | All legs drawn; stops numbered 1, 2, 3 | Each leg visible; numbers match the stop list |
| 11 | **Open in Google Maps** on leg 2 | Google Maps opens with B→C, transit mode | The part card shows the time to leave, since the link can't carry it |
| 12 | **Something changes.** Change B's stay to 15 min and re-plan | Leg 2 and the summary update | No old results left on screen |
| 13 | Extension: open google.com/maps, search a place, open the side panel, press **Add this place** | The place is added as the next stop in the app | Name and position match the Maps page |
| 14 | Extension on a non-place page | "Open a place in Google Maps first." | No stop added, no error in console |
| 15 | Phone at 200% text size | Still readable, buttons still tappable | No overlap, no sideways scrolling |
| 16 | Midnight: start 11:00 PM, stay 90 min at B | Leg 2 leaves the next day; the card shows the date | Times after midnight are on the right day |

## 4. Functional requirements

13 Must, 3 Should, 2 Could. Should and Could work starts only after every Must passes its walkthrough row.

| ID | Area | Requirement | Priority |
|---|---|---|---|
| R1 | Stops | Enter 2–8 stops (**PROPOSED** max 8) by typing, with Google place suggestions | Must |
| R2 | Stops | Pick trip date (today to +100 days) and start time (default: now, rounded up to 5 min) | Must |
| R3 | Stops | Each middle stop: choose **Leave at [time]** or **Stay for [minutes]** (1–720) | Must |
| R4 | Plan | Plan each leg by transit, one Google request per leg, departing at that stop's leave time | Must |
| R5 | Show | Per leg: each walk and transit step with line, direction, board/exit stop, depart/arrive times, stop count | Must |
| R6 | Show | Trip end time and total trip length | Must |
| R7 | Show | Fare per leg and total fare; "Fare not available" where Google has none; totals per currency | Must |
| R8 | Warn | Warn when arrival is after a "Leave at" time; the next leg uses the real arrival | Must |
| R9 | Warn | Clear message when a leg has no transit route; later legs aren't planned | Must |
| R10 | Access | Large plain-language layout for all ages; phone and desktop; meets WCAG 2.2 AA | Must |
| R11 | Show | Map showing every leg and numbered stops | Must |
| R12 | Show | "Open in Google Maps" button per leg | Should |
| R13 | Extension | Chrome side panel shows the web app | Must |
| R14 | Extension | "Add this place" adds the place open on google.com/maps as a stop | Must |
| R15 | Stops | Remove a stop; move a stop up or down | Should |
| R16 | Stops | Remember the last trip on this device | Should |
| R17 | Share | Share a trip as a link | Could |
| R18 | Share | Print-friendly trip sheet | Could |

## 5. Architecture

**The insight:** Google won't route transit through stops in between, so the app chains one-leg requests, using each stop's leave time as the next leg's departure.

```
 ┌─ Chrome extension (local files) ──────────────────────────┐
 │ side panel page ── iframe ──┐   "Add this place" button   │
 │                             │      reads google.com/maps   │
 │                             │      tab URL → mapsurl.js    │
 │                             │      → postMessage(stop) ────┼──┐
 └─────────────────────────────┼──────────────────────────────┘  │
                               ▼                                 ▼
 ┌─ Web app (static files, GitHub Pages) ───────────────────────────┐
 │ app.js (screen) ──► trip.js (pure: times, fares, text)           │
 │      │                  ▲  planTrip() calls computeLeg per leg   │
 │      ▼                  │                                        │
 │ maps.js ── the only file that talks to Google ───────────────────┼─► Google Maps JS API
 └──────────────────────────────────────────────────────────────────┘    Autocomplete · Route.computeRoutes · Map
```

In words: the user types stops in **app.js**. **trip.js** works out each leg's departure and calls `computeLeg` once per leg, in order. `computeLeg` is in **maps.js**, which asks Google and returns plain data. trip.js then adds up times and fares, and app.js shows the result and asks maps.js to draw the map. The extension's side panel holds the same web app in an iframe. Its button turns the Google Maps tab's address into a stop and sends it to the app.

**Trust boundary:** only maps.js sends anything off the device, and only to Google. The app accepts an extension message only when it comes from the parent frame, from a `chrome-extension://` origin, and has a valid shape (see [context/contracts.md](../context/contracts.md)).

## 6. Tech stack

| Layer | Choice | Why | Fallback |
|---|---|---|---|
| App | Plain HTML, CSS, JavaScript ES modules; no framework, no build step | Easiest to read and change; one screen doesn't need more | — |
| Place search | Maps JS `PlaceAutocompleteElement`, `gmp-select` event → `placePrediction.toPlace()` → `fetchFields(['displayName','location','id'])` ([docs](https://developers.google.com/maps/documentation/javascript/place-autocomplete-new)) | Google's own suggestions, worldwide | None needed separately: it loads with the rest of Google Maps, so if it fails the app shows the "Can't reach Google Maps" banner |
| Routing | Maps JS `Route.computeRoutes`, loaded with `importLibrary('routes')`, `travelMode: 'TRANSIT'`, `departureTime`, `fields: ['path','legs','travelAdvisory','localizedValues']` ([docs](https://developers.google.com/maps/documentation/javascript/routes/route-transit)) | Replaces the legacy DirectionsService ([overview](https://developers.google.com/maps/documentation/javascript/routes/route-class-overview)); has transit details and fares | Routes REST API via `fetch` (same data) |
| Map | Maps JS Map; `route.createPolylines()` per leg ([docs](https://developers.google.com/maps/documentation/javascript/routes/get-a-route)) | Draws transit segments for us | Hide the map; the leg list still works |
| Hosting | GitHub Pages at `https://aokhader.github.io/NonStopRoutes/`, deployed by GitHub Actions | Free, same repo | Cloudflare Pages or Netlify |
| Extension | Chrome Manifest V3 side panel, `"sidePanel"` permission, Chrome 114+ ([docs](https://developer.chrome.com/docs/extensions/reference/api/sidePanel)) | Same app, no second codebase | Toolbar button opens the app in a new tab |
| Tests | `node --test` (Node 22+, installed: 22.23.1) | Built in, zero dependencies | — |
| Local server | `python -m http.server 8000 -d web` (Python installed: 3.14) | ES modules and the API key need `http://`, not `file://` | `npx --yes http-server web -p 8000` |

**Limits checked in docs:**
- Transit departure time can be up to 7 days in the past and 100 days in the future ([transit route](https://developers.google.com/maps/documentation/javascript/routes/route-transit)). Hence R2's +100 days.
- Transit can't use intermediate waypoints, traffic settings or eco routes (same page).
- Fare appears only when Google knows the fare for **every** transit step in the route ([transit route](https://developers.google.com/maps/documentation/routes/transit-route)).
- Compute Routes rate limit: 3,000 requests/minute ([usage and billing](https://developers.google.com/maps/documentation/routes/usage-and-billing)).
- Maps URLs (`https://www.google.com/maps/dir/?api=1&origin=…&destination=…&travelmode=transit`) have **no departure-time parameter** and a 2,048-character limit ([Maps URLs](https://developers.google.com/maps/documentation/urls/get-started)).
- Billing tiers: Essentials covers requests with basic features only, and Pro covers advanced features such as traffic-aware routing ([usage and billing](https://developers.google.com/maps/documentation/routes/usage-and-billing)). Each Essentials SKU has 10,000 free requests/month and each Pro SKU 5,000 ([pricing overview](https://developers.google.com/maps/billing-and-pricing/overview)). **ASSUMPTION:** a transit request bills as Essentials. Check the billing report after Phase 3.
- Reading a tab's URL needs the `"tabs"` permission or host permissions for that site ([tabs API](https://developer.chrome.com/docs/extensions/reference/api/tabs)). We use host permission `https://www.google.com/maps/*`.
- **ASSUMPTION:** a side-panel page may iframe a remote https page, and GitHub Pages allows framing. The MV3 default CSP sets only `script-src` and `object-src` ([CSP docs](https://developer.chrome.com/docs/extensions/reference/manifest/content-security-policy)), but neither point is stated outright. Check in Phase 8.
- **ASSUMPTION:** the Cloud project needs **Maps JavaScript API**, **Places API (New)** and **Routes API** enabled. Confirm in Phase 1/3: an "API not activated" error names the missing one.

### Setup checklist
Steps 1–6: before any P1 code. Step 7: during P1, once `config.example.js` exists. Step 8: in P7.
1. Sign in at [console.cloud.google.com](https://console.cloud.google.com) → create project `nonstoproutes`.
2. Billing → create a billing account (card needed) → link it to the project.
3. APIs & Services → Enable: Maps JavaScript API, Places API (New), Routes API.
4. Credentials → Create API key → **Website restrictions:** `http://localhost:8000/*`, `https://aokhader.github.io/NonStopRoutes/*` → **API restrictions:** the 3 APIs above.
5. Quotas → cap requests per day per API (**PROPOSED**: Routes 500, Places 1,000, Maps loads 500).
6. Billing → Budgets & alerts → $5/month budget, email at 50%, 90%, 100%.
7. Copy `web/config.example.js` to `web/config.js` and paste the key. `config.js` is gitignored.
8. GitHub repo → Settings → Secrets and variables → Actions → add `MAPS_API_KEY`. Settings → Pages → Source: **GitHub Actions**.

## 7. Logic, formulas, and test cases

**Leave time at each stop** (stop 1 leaves at the trip start; the last stop has no leave rule):

```
Stay for:   leave  = arrive + stay minutes
Leave at:   wanted = the moment whose clock reads hh:mm that is NEAREST to arrive (within 12 h either side)
            leave  = the later of arrive and wanted
            warning (R8) when arrive is later than wanted

trip length = arrival at the last stop − trip start
fare milli  = units × 1000 + round(nanos ÷ 1,000,000)
```

- "Nearest to arrival" keeps "Leave at 2:30 PM" on the same afternoon when you arrive at 2:40 PM (→ warning). It moves "12:30 AM" to the next day when you arrive at 11:40 PM. And it turns a too-early time (12:00 PM after a 1:40 PM arrival) into a warning instead of a silent next-day plan. shortcut: stays over 12 hours aren't supported, upgrade if users need overnight stops.
- Fare: Google gives `units` + `nanos` (billionths). Store **thousandths** of the main currency unit as a whole number, which is exact for currencies with up to 3 decimals (USD 2, KWD 3, JPY 0). Sum per currency.
- Times are shown in the browser's time zone (shortcut, see Risks).

**Test cases.** They go in `tests/` and use fixed dates, so every run gives the same result. Times are local, 2026-11-22 unless marked.

| ID | Function | Input | Expected |
|---|---|---|---|
| T1 | resolveClock | "14:30", near 14:00 | 14:30 same day |
| T2 | resolveClock | "00:30", near 23:40 | 00:30 on 11-23 |
| T3 | resolveClock | "12:00", near 13:40 | 12:00 same day (the earlier one is nearer) |
| T4 | leaveTime | stay 45, arrive 09:31 | 10:16, no warning |
| T5 | leaveTime | stay 0, arrive 09:31 | throws `RangeError` (stay must be 1–720) |
| T6 | leaveTime | leave-at 14:30, arrive 14:10 | 14:30, no warning |
| T7 | leaveTime | leave-at 14:30, arrive 14:40 | 14:40, warning {arriveAt 14:40, wantedAt 14:30} |
| T8 | leaveTime | leave-at 14:30, arrive 14:30 | 14:30, no warning |
| T9 | leaveTime | stay 90, arrive 23:10 | 00:40 on 11-23 |
| T10 | planTrip | A→B→C, start 09:00, B stay 45, fake leg = 30 min | leg1 09:00–09:30, leg2 10:15–10:45 |
| T11 | planTrip | 3 legs, fake returns `null` for leg 2 | returns 2 legs: `ok`, `noRoute`; leg 3 never requested |
| T12 | planTrip | 2 stops | exactly 1 computeLeg call |
| T13 | planTrip | fake throws on leg 1 | returns 1 leg, status `error`; no further calls |
| T14 | summarize | leave 09:00, arrive 10:45 | totalMinutes 105 |
| T15 | summarize | fares USD 2750 + USD 2750 | totals [USD 5500], missing [] |
| T16 | summarize | USD 2750 + none | totals [USD 2750], missing [2] |
| T17 | summarize | none + none | totals [], missing [1, 2] |
| T18 | summarize | USD 2750 + EUR 1500 | totals [USD 2750, EUR 1500] |
| T19 | toMilli | units "2", nanos 750000000 | 2750 |
| T20 | toMilli | units "0", nanos 250000000 (KWD) | 250 |
| T21 | formatDuration | 45 / 60 / 125 | "45 min" / "1 h" / "2 h 5 min" |
| T22 | formatFare | USD 2750 / KWD 250 / JPY 210000 | "$2.75" / "KWD 0.250" / "¥210" (en-US `Intl.NumberFormat`) |
| T23 | mapsLink | from place id, to lat/lng | `https://www.google.com/maps/dir/?api=1&origin=…&origin_place_id=…&destination=lat,lng&travelmode=transit` |
| T24 | readAddStopMessage | `{type:'nsr:add-stop', name:'Cafe', lat:24.7, lng:46.6}` | stop `{name:'Cafe', lat:24.7, lng:46.6}` |
| T25 | readAddStopMessage | lat 120, or name of 201 chars, or wrong type | `null` |
| T26 | parseMapsUrl | `…/maps/place/Joe%27s+Cafe/@40.71,-74.00,17z/data=…!3d40.7128!4d-74.0060…` | `{name:"Joe's Cafe", lat:40.7128, lng:-74.006}` (pin, not viewport) |
| T27 | parseMapsUrl | place URL with `@lat,lng` but no `!3d!4d` | lat/lng from `@` |
| T28 | parseMapsUrl | `…/maps/search/…` or `…/maps/dir/…` | `null` |
| T29 | parseMapsUrl | `https://example.com/maps/place/x/@1,2,3z` | `null` |
| T30 | formatTime | 09:07 / 21:07 | "9:07 AM" / "9:07 PM" |
| T31 | defaultStart | now 09:02 / 09:05:00 / 09:05:30 / 23:58 | 09:05 / 09:05 / 09:10 / 00:00 on 11-23 |
| T32 | tripDayProblem | now 2026-11-22 10:00; start 11-22 09:00 / 2027-03-02 09:00 / 2027-03-02 11:00 / 11-21 09:00 | `null` / `null` / "Pick a day from today up to 100 days ahead." / same message |

## 8. Build phases

You're building alone with no deadline, so the plan is a sequence of phases. Each one ends with something you can run. Details and first tasks are in [context/progress.md](../context/progress.md).

| Phase | Builds | Requirements | Done when |
|---|---|---|---|
| P1 Setup | Cloud checklist, `.gitignore`, `package.json`, `web/index.html` showing a map | — | A map loads at `http://localhost:8000` |
| P2 Logic | `web/trip.js` + `tests/trip.test.js` | R2, R3, R6, R7, R8 logic; helpers for R12, R14 | T1–T25 and T30–T32 pass with `npm test` |
| P3 One leg | `maps.js` (place picker, computeLeg); 2 stops → 1 part card | R1 (2 stops), R2, R4, R5 | Row 1 passes, and row 4 passes for a 2-stop trip |
| P4 Whole trip | Up to 8 stops, leave rules, summary, fares, warnings, no-route | R1, R3, R6–R9 | Rows 2–9, 12, 16 pass |
| P5 Map | Map drawing, numbered stops, Open in Google Maps | R11, R12 | Rows 10–11 pass |
| P6 Readable | Accessibility pass, remove/move stops, remember last trip | R10, R15, R16 | Rows 1, 15 pass; a real older user plans a trip unaided |
| P7 Deploy | GitHub Actions → Pages, key restricted to the Pages URL | — | Walkthrough passes on the live URL |
| P8 Extension | `extension/` side panel, `mapsurl.js` + tests | R13, R14 | Rows 13–14 pass; T26–T29 pass |
| Later | Share link, print sheet | R17, R18 | — |

## 9. Risks and fallbacks

| Risk | Fallback |
|---|---|
| Google changes prices or free caps | Daily quota caps + $5 budget alert (setup steps 5–6) |
| Transit bills as Pro, not Essentials (**ASSUMPTION**) | 5,000 free/month is still plenty for personal use; check the billing report after P3 |
| Many cities have no fare data | Show "Fare not available" per leg; never guess or show $0 |
| API key copied from the page source and misused | Website + API restrictions, quotas, budget alert. A browser key is public by design, so restrictions are the protection |
| Google changes its Maps URL format, breaking "Add this place" | `parseMapsUrl` returns `null` → "Open a place in Google Maps first"; fix the parser against T26–T29 |
| Only `www.google.com/maps` is supported (not `google.co.uk/maps` etc.) | shortcut; add more host permissions if users need them |
| GitHub Pages or Chrome blocks the iframe in the side panel | Toolbar button opens the app in a new tab instead (R13 then = "opens the app") |
| Times shown in the browser's time zone, not the trip city's | shortcut; fine when planning where you live. Upgrade: use Google's localized time text |
| Real buses run late or schedules change | Note on results: "Times come from Google and can change. Check before you go." |
| Older users get lost in options | One screen, defaults set, plain words; P6 test with a real older user |
| Legacy Google APIs (DirectionsService, legacy Autocomplete) | Not used; new projects should use the Route class and PlaceAutocompleteElement |

## 10. Open questions

- [ ] English only for now? (**ASSUMPTION**: yes)
- [ ] Max 8 stops OK? (**PROPOSED**)
- [ ] New middle stops default to "Stay for 30 min"? (**PROPOSED**)
- [ ] Repo public? GitHub Pages on a private repo needs a paid GitHub plan.
- [ ] Publish the extension on the Chrome Web Store (one-time developer fee, **ASSUMPTION** $5), or keep it as a load-unpacked extension for now?
- [ ] Is the time-zone shortcut OK (plans shown in your own time zone)?
- [ ] Daily quota numbers in setup step 5 OK? (**PROPOSED**)

## Sources
- [Routes API: get a transit route](https://developers.google.com/maps/documentation/routes/transit-route)
- [Compute routes (transit + intermediates)](https://developers.google.com/maps/ai/maps-experience-agent-tools/compute-routes)
- [Maps JS: get a transit route](https://developers.google.com/maps/documentation/javascript/routes/route-transit)
- [Maps JS: get a route](https://developers.google.com/maps/documentation/javascript/routes/get-a-route)
- [Maps JS: Route class overview](https://developers.google.com/maps/documentation/javascript/routes/route-class-overview)
- [Maps JS: Route reference](https://developers.google.com/maps/documentation/javascript/reference/route)
- [Maps JS: Place Autocomplete widget](https://developers.google.com/maps/documentation/javascript/place-autocomplete-new)
- [Routes API usage and billing](https://developers.google.com/maps/documentation/routes/usage-and-billing)
- [Google Maps Platform pricing overview](https://developers.google.com/maps/billing-and-pricing/overview)
- [Maps URLs](https://developers.google.com/maps/documentation/urls/get-started)
- [chrome.sidePanel](https://developer.chrome.com/docs/extensions/reference/api/sidePanel)
- [chrome.tabs](https://developer.chrome.com/docs/extensions/reference/api/tabs)
- [Extension content security policy](https://developer.chrome.com/docs/extensions/reference/manifest/content-security-policy)
