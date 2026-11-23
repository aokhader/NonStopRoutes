# Setup

Everything needed to run, test, deploy and load NonStopRoutes on Windows.

## 1. Tools (already installed on this machine)
| Tool | Version | Used for |
|---|---|---|
| Node | 22.23.1 | `npm test` (built-in test runner) |
| Python | 3.14 | Local web server |
| Chrome | 114+ | Side panel extension |
| Git | any | — |

## 2. Google Cloud (Phase 1, once)
Follow the **Setup checklist** in [SPEC §6](../docs/SPEC.md#setup-checklist): steps 1–6 before coding, step 7 during P1, step 8 in P7. Summary:
- Project `nonstoproutes`, with billing linked.
- Enable **Maps JavaScript API**, **Places API (New)**, **Routes API**.
- One API key. Website restrictions: `http://localhost:8000/*` and `https://aokhader.github.io/NonStopRoutes/*`. API restrictions: those 3 APIs.
- Daily quota caps and a $5 budget alert.

If the browser console says an API "has not been used in project … or it is disabled", enable the API it names, wait 2 minutes, and reload.

## 3. Key file
`web/config.example.js` (committed):
```js
// Copy to web/config.js and paste your browser key. config.js is gitignored.
export const MAPS_API_KEY = 'PASTE-YOUR-KEY-HERE';
```
`web/config.js` (never committed) has the same line with the real key.

`.gitignore`:
```
web/config.js
.env
node_modules/
```

## 4. Run and test
```
python -m http.server 8000 -d web
```
Open http://localhost:8000. It must be port 8000, because the key only allows it.
```
npm test
```

## 5. Deploy (Phase 7)
- GitHub → repo **Settings → Secrets and variables → Actions** → New secret `MAPS_API_KEY`.
- **Settings → Pages → Source: GitHub Actions.** Pages on a private repo needs a paid GitHub plan.
- `.github/workflows/pages.yml`, on push to `main`:
  1. Check out the repo.
  2. Write `web/config.js` from `secrets.MAPS_API_KEY`.
  3. Upload `web/` as the Pages artifact.
  4. Deploy to Pages.
  Use GitHub's current `upload-pages-artifact` / `deploy-pages` actions, and check their latest versions in GitHub docs when writing the workflow.
- Live at https://aokhader.github.io/NonStopRoutes/

## 6. Load the extension (Phase 8)
1. Chrome → `chrome://extensions` → turn on **Developer mode**.
2. **Load unpacked** → pick the `extension/` folder.
3. Pin it. Clicking its icon opens the side panel.
4. For local testing, set `APP_URL` in `extension/sidepanel.js` to `http://localhost:8000/`. Set it back to the Pages URL before sharing.
5. After editing extension files, press **Reload** on its card in `chrome://extensions`.

## 7. Smoke check per phase
| Phase | Check |
|---|---|
| P1 | http://localhost:8000 shows a map; no red errors in DevTools console |
| P2 | `npm test`: T1–T25 and T30–T32 pass |
| P3 | Two real stops → one part card with walk + bus/train steps (walkthrough #1, #4) |
| P4 | 3+ stops → summary with end time and fare; late warning appears when expected |
| P5 | Map shows every part; Open in Google Maps opens the right part |
| P6 | Phone at 200% text: walkthrough #1 and #15 pass |
| P7 | Walkthrough passes on the live URL |
| P8 | Side panel shows the app; Add this place adds the place; T26–T29 pass |
