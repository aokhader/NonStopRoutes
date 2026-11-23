# Contracts

Every data shape and exported function that crosses a file boundary. **Rule:** to change one, edit this file first, add a line to [decisions.md](decisions.md), then change the code and its tests.

All shapes are plain objects. Times are JavaScript `Date` objects (absolute moments), shown in the browser's time zone. Write these as JSDoc in the code; no TypeScript.

## Shapes
```js
/** A place the user picked. placeId is missing when it came from the extension. */
/** @typedef {{ name: string, lat: number, lng: number, placeId?: string }} Place */

/** How a middle stop is left. time is 24-hour "HH:MM". minutes is 1–720. */
/** @typedef {{ type: 'at', time: string } | { type: 'stay', minutes: number }} LeaveRule */

/** First stop: no leave rule (it leaves at trip.start). Last stop: no leave rule. */
/** @typedef {Place & { leave?: LeaveRule }} Stop */

/** @typedef {{ start: Date, stops: Stop[] }} Trip          // 2–8 stops */

/** Fare in thousandths of the main currency unit: $2.75 → { currency: 'USD', milli: 2750 }. */
/** @typedef {{ currency: string, milli: number }} Fare */

/** One line in a part card. Transit-only fields are missing for WALK. */
/** @typedef {{
 *   mode: 'WALK' | 'TRANSIT',
 *   minutes: number,
 *   line?: string,        // "12" or "Red Line": short name, else name
 *   vehicle?: string,     // "Bus", "Train", "Subway", "Tram" …
 *   headsign?: string,    // direction, e.g. "Downtown"
 *   boardStop?: string, exitStop?: string,
 *   departAt?: Date, arriveAt?: Date,
 *   stopCount?: number    // stops ridden, counting the exit stop
 * }} Step */

/** What computeLeg returns. departAt = when Google says to set off (can be a few minutes after readyAt).
 *  googleRoute is "opaque": other files pass it along but never look inside; only maps.js reads it. */
/** @typedef {{ departAt: Date, arriveAt: Date, steps: Step[], fare: Fare | null, googleRoute: object }} LegData */

/** One leg after planning.
 *  readyAt = when the user can leave `from` (trip start, or the stop's leave rule); sent to Google as departureTime.
 *  warning = the user arrives at `from` later than its "Leave at" time. */
/** @typedef {{
 *   from: Stop, to: Stop,
 *   status: 'ok' | 'noRoute' | 'error',
 *   readyAt: Date,
 *   warning?: { arriveAt: Date, wantedAt: Date },
 *   data?: LegData        // only when status is 'ok'
 * }} Leg */

/** complete = every leg is 'ok'. endAt/totalMinutes are null when not complete. */
/** @typedef {{
 *   startAt: Date,             // legs[0].readyAt, i.e. the trip start
 *   endAt: Date | null, totalMinutes: number | null,
 *   totals: Fare[],            // one per currency, in first-seen order
 *   missingFareLegs: number[], // 1-based numbers of 'ok' legs whose fare is null
 *   complete: boolean
 * }} Summary */
```

## web/trip.js (pure, every function tested)
| Function | Returns | Notes | Tests |
|---|---|---|---|
| `resolveClock(hhmm, near)` | `Date` | The moment whose local clock reads `hhmm` nearest to `near` (within 12 h either side) | T1–T3 |
| `leaveTime(rule, arriveAt)` | `{ leaveAt: Date, warning? }` | Throws `RangeError` if stay isn't 1–720 | T4–T9 |
| `planTrip(trip, computeLeg, onProgress?)` | `Promise<Leg[]>` | One leg after another. Stops at the first `noRoute`/`error` and returns the legs so far, including the failed one. `onProgress(i, n)` before each leg | T10–T13 |
| `summarize(trip, legs)` | `Summary` | Sums `milli` per currency over `ok` legs | T14–T18 |
| `toMilli(units, nanos)` | `number` | `units` may arrive as text, because Google sends very large whole numbers ("int64") as strings | T19–T20 |
| `formatTime(date)` | `"9:07 AM"` | `Intl.DateTimeFormat('en-US', {hour:'numeric', minute:'2-digit'})` | T30 |
| `formatDuration(minutes)` | `"2 h 5 min"` | | T21 |
| `formatFare(fare)` | `"$2.75"` | `Intl.NumberFormat('en-US', {style:'currency', currency})`. Some currencies come out with a non-breaking space (character U+00A0), so tests swap it for a normal space before comparing | T22 |
| `defaultStart(now)` | `Date` | `now` rounded **up** to the next 5 minutes (unchanged if already on one) | T31 |
| `tripDayProblem(start, now)` | `string \| null` | `null` if `start` is from the start of today up to `now` + 100 days (add days with `setDate`, not milliseconds); else the message | T32 |
| `mapsLink(from, to)` | `string` | `https://www.google.com/maps/dir/?api=1&origin=…&destination=…&travelmode=transit`. Uses `name` + `*_place_id` when `placeId` exists, else `lat,lng` | T23 |
| `readAddStopMessage(data)` | `Place \| null` | Checks the extension message below | T24–T25 |

**Why `planTrip` takes `computeLeg` as an argument:** `planTrip` must never call Google itself, or `trip.js` couldn't be tested without internet. So `app.js` hands it the real `computeLeg` from `maps.js`, and the tests hand it a fake one (e.g. "every leg takes 30 min"). Same code, different helper.

`computeLeg`'s shape: `(from: Place, to: Place, readyAt: Date) => Promise<LegData | null>`.

## web/maps.js (only Google caller)
| Function | Returns | Notes |
|---|---|---|
| `loadGoogle(apiKey)` | `Promise<void>` | Loads the Maps JS API once; rejects with a plain `Error` on failure |
| `createMap(mapEl)` | map object | Shows a world view (zoom 2) until a trip is drawn. No location prompt |
| `attachPlacePicker(containerEl, onPick)` | `void` | Puts a `PlaceAutocompleteElement` in `containerEl`; on `gmp-select` → `toPlace()` → `fetchFields(['displayName','location','id'])` → `onPick(Place)` |
| `computeLeg(from, to, readyAt)` | `Promise<LegData \| null>` | One `Route.computeRoutes` call, `travelMode: 'TRANSIT'`, `departureTime: readyAt`, `fields: ['path','legs','travelAdvisory','localizedValues']`. `null` = no route. Throws on error/timeout. **Check exact response field names in the Route reference before writing this** |
| `drawTrip(map, legs)` | `void` | Clears the old drawing; `createPolylines()` per ok leg; numbered markers 1…n; fits bounds |

## Extension → app message
```js
{ type: 'nsr:add-stop', name: 'Corner Pharmacy', lat: 40.7128, lng: -74.006 }
```
| Field | Rule |
|---|---|
| `type` | Exactly `'nsr:add-stop'` |
| `name` | String, 1–200 characters after trimming |
| `lat` | Finite number, −90…90 |
| `lng` | Finite number, −180…180 |

Receiver (`app.js`) also requires `event.source === window.parent` and `event.origin.startsWith('chrome-extension://')`. Any extension may send it, which is acceptable because the worst it can do is add a stop the user can see and remove.

## extension/mapsurl.js (pure)
| Function | Returns | Notes | Tests |
|---|---|---|---|
| `parseMapsUrl(url)` | `Place \| null` | Only `https://www.google.com/maps/place/<name>/…`. Pin from `!3d<lat>!4d<lng>`, else `@<lat>,<lng>`. Name: URL-decoded, `+` → space | T26–T29 |

## Saved trip (R16, localStorage)
Key `nsr:lastTrip`, value JSON:
```json
{ "version": 1, "date": "2026-11-22", "time": "09:00",
  "stops": [ { "name": "Home", "lat": 40.71, "lng": -74.0, "placeId": "abc" },
             { "name": "Pharmacy", "lat": 40.72, "lng": -73.99, "leave": { "type": "stay", "minutes": 45 } },
             { "name": "Library", "lat": 40.73, "lng": -73.98 } ] }
```
Wrap reads and writes in `try/catch`. Anything that doesn't parse, or has the wrong `version`, is ignored and the app starts empty.
