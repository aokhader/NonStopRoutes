# UI

One screen. The user may be 15 or 85, on a small phone or in a 360px side panel. When in doubt, make it bigger and say it plainer.

## Layout (top to bottom)
1. **Title:** "Plan a trip with stops"
2. **When:** "Day" (date input, today…+100 days) and "Start time" (time input, default now rounded up to 5 min)
3. **Stops:** numbered cards
   - **1 · Start:** place box only (it leaves at the start time)
   - **2…n−1 · Stop:** place box + a big two-choice control: **Leave at** [time] / **Stay for** [minutes]. New stops default to **Stay for 30 min** (**PROPOSED**). Buttons: Remove, Move up, Move down (R15)
   - **n · Final stop:** place box only
4. **+ Add a stop** (hidden at 8 stops; says "Up to 8 stops")
5. **Plan my trip:** full-width primary button
6. **Results** (after planning): summary card → one card per part → map → note "Times come from Google and can change. Check before you go."

## States
| State | What shows | Driven by |
|---|---|---|
| empty | Form with 2 blank stops, Plan button | Page load, no saved trip |
| editing | Form; Plan button enabled once every stop is picked | Any input |
| planning | Form disabled; "Planning part 2 of 4…" in a live region | Plan my trip |
| results | Summary + all part cards + map | All legs `ok` |
| partial | Cards up to the failed part; failed card shows the message; summary says "Trip couldn't be fully planned" | A leg `noRoute`/`error` |
| failed | Banner: "Can't reach Google Maps. Check your internet, then press Try again." | `loadGoogle` rejected |

## Words on screen
Use "part" on screen for a leg ("Part 1: Home → Pharmacy"). Code says `leg`. (**PROPOSED**)

| Thing | Exact text pattern |
|---|---|
| Summary | "Trip ends **11:02 AM** · 2 h 2 min" / "Total fare **$5.50**" |
| Fare unknown, some | "At least $2.75 (fare unknown for part 2)" |
| Fare unknown, all | "Fare not available" |
| Part header | "Part 1: Home → Pharmacy" / "Leave 9:02 AM · Arrive 9:31 AM · 29 min · $2.75". Leave/Arrive = `data.departAt` / `data.arriveAt` (Google's times) |
| Walk step | "Walk 5 min to Main St & 3rd". The place = the next step's `boardStop`, or the part's destination for the last walk |
| Transit step | "**Bus 12** toward Downtown. Get on at Main St & 3rd at 9:07 AM. Ride 6 stops. Get off at Oak Ave (9:25 AM)" |
| Late warning | "⚠ Warning: You'll arrive at 9:31 AM, after your 9:20 AM leave time. Part 2 is planned from 9:31 AM." |
| No route | "No bus or train found for this part at this time. Try another time or stop." |
| Leg error | "Couldn't plan this part. Check your internet and press Plan my trip again." |
| Other day | Add the day when it isn't the trip's first day: "Mon, Nov 23 · 12:40 AM" |
| Extension, wrong page | "Open a place in Google Maps first." |

Rules: short sentences; the most important fact first; times like "9:07 AM", never "09:07" or "21:07"; durations like "1 h 5 min"; no jargon ("waypoint", "transit mode", "polyline", "API").

## Accessibility (WCAG 2.2 AA, R10)
- Body text **20px** (**PROPOSED**, never below 18px), line height 1.5, system font stack.
- Text contrast ≥ 4.5:1, large text and icons ≥ 3:1. Check every color pair with a contrast checker.
- Tap targets ≥ **48×48 px**, with ≥ 8px between them.
- Every input has a visible `<label>`; the two-choice control is real radio buttons (`<fieldset>` + `<legend>`).
- Visible focus outline (3px). Everything works with the keyboard alone, in a logical tab order.
- `aria-live="polite"` region for "Planning part…" and for the summary when results arrive.
- **Never color alone:** warnings have an icon *and* the word "Warning"; map parts have numbered markers *and* the list.
- The map is extra: every fact on it is also in the text.
- Works at 320px wide and at 200% text zoom with no sideways scrolling. Single column, `max-width: 40rem`.
- `<html lang="en">`. No timeouts that hide things, no animation.

## Colors (PROPOSED, check contrast before use)
| Use | Color |
|---|---|
| Page / text | `#FFFFFF` / `#1A1A1A` |
| Primary button | `#0B57D0` background, `#FFFFFF` text |
| Warning box | `#FFF4E5` background, `#7A4100` text, ⚠ icon |
| Error box | `#FDECEA` background, `#8C1D18` text |
| Map parts | A different color per part, plus numbered markers |
