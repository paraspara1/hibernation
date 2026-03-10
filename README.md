# Slice Hibernation Config

This repository is the remote configuration source for the Slice mobile app during its hibernation period.

After Day 0 (when backend infrastructure is decommissioned), this repo is the **only mechanism** for changing app behaviour without an App Store release. The app fetches `hibernation-config.json` on every launch. If the fetch succeeds, values are stored on-device and used. If the fetch fails, the app falls back to the last cached values, then to hardcoded defaults.

## How to make a change

1. Edit `hibernation-config.json` directly in the GitHub web UI (no local setup needed)
2. Commit to `main`
3. Users will pick up the change on their **next app open** (subject to a 5-second network timeout)

---

## Config reference

```json
{
  "hibernationStartAt": "2026-03-09T00:00:00.000Z",
  "closureDate": "31 May 2026",
  "portOutDeadline": "31 May 2026",
  "environments": {
    "dev": true,
    "staging": true,
    "production": true
  },
  "links": {
    "home": "https://www.slicemobile.com",
    "terms": "https://www.slicemobile.com/terms-conditions",
    "privacy": "https://www.slicemobile.com/privacy-policy",
    "closureFaq": "https://help.slicemobile.com/en/",
    "dataRequest": "https://www.slicemobile.com/privacy-policy"
  }
}
```

| Field | Type | Description |
|---|---|---|
| `hibernationStartAt` | ISO datetime string | When the hibernation flow activates. The app computes `hibernationEnabled = now >= hibernationStartAt`. Set this in advance — the app self-activates at the right moment without needing a midnight push. |
| `closureDate` | string | The date shown to users as the service end date (e.g. `"31 May 2026"`). |
| `portOutDeadline` | string | The deadline shown to users for porting their number out (e.g. `"31 May 2026"`). |
| `environments.dev` | boolean | Whether the hibernation flow is active in dev builds. |
| `environments.staging` | boolean | Whether the hibernation flow is active in staging builds. |
| `environments.production` | boolean | Whether the hibernation flow is active in production builds. |
| `links.home` | string | Main website URL. Update this when `slicemobile.com` goes offline. |
| `links.terms` | string | Terms & Conditions URL. |
| `links.privacy` | string | Privacy Policy URL. |
| `links.closureFaq` | string | FAQ page for the closure. Update this when the help site goes offline. |
| `links.dataRequest` | string | URL for users to submit a data request. |

---

## Common scenarios

**Activating hibernation in production:**
Set `hibernationStartAt` to the intended activation datetime in ISO format (e.g. `"2026-04-01T08:00:00.000Z"`). Commit whenever you like — the app will self-activate when the clock passes that time. No midnight push needed.

**Testing the hibernation flow before going live:**
Set `environments.production` to `false` and `environments.dev`/`environments.staging` to `true`. Dev and staging builds will show the hibernation flow; production users will not be affected.

**The closure date changes:**
Update `closureDate` and/or `portOutDeadline` and commit.

**`slicemobile.com` goes offline:**
Update all affected `links.*` fields to point at alternative URLs (e.g. GitHub Pages at `paraspara1.github.io/...`) and commit.

**Funding is secured and the app should return to normal:**
Set `hibernationStartAt` to a far-future date (e.g. `"2099-01-01T00:00:00.000Z"`). Users who open the app after that will bypass the hibernation flow. A full App Store update will still be needed to restore all normal functionality.

---

## Fallback behaviour

The app resolves config in this order on every launch:

1. **Remote** — this file, fetched over the network (5-second timeout)
2. **On-device cache** — the last successfully fetched config, stored locally
3. **Hardcoded defaults** — baked into the app binary at build time

If this repository or GitHub itself becomes unreachable, users will continue to see the last config they successfully fetched.
