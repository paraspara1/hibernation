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
  "hibernationEnabled": true,
  "closureDate": "31 May 2026",
  "portOutDeadline": "31 May 2026",
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
| `hibernationEnabled` | boolean | Controls whether the app shows the hibernation flow on launch. Set to `false` if funding is secured and normal service resumes. |
| `closureDate` | string | The date shown to users as the service end date (e.g. `"31 May 2026"`). |
| `portOutDeadline` | string | The deadline shown to users for porting their number out (e.g. `"31 May 2026"`). |
| `links.home` | string | Main website URL. Update this when `slicemobile.com` goes offline. |
| `links.terms` | string | Terms & Conditions URL. |
| `links.privacy` | string | Privacy Policy URL. |
| `links.closureFaq` | string | FAQ page for the closure. Update this when the help site goes offline. |
| `links.dataRequest` | string | URL for users to submit a data request. |

---

## Common scenarios

**The closure date changes:**
Update `closureDate` and/or `portOutDeadline` and commit.

**`slicemobile.com` goes offline:**
Update all affected `links.*` fields to point at alternative URLs (e.g. GitHub Pages at `paraspara1.github.io/...`) and commit.

**Funding is secured and the app should return to normal:**
Set `hibernationEnabled` to `false` and commit. Users who open the app after that will bypass the hibernation flow. A full App Store update will still be needed to restore all normal functionality.

---

## Fallback behaviour

The app resolves config in this order on every launch:

1. **Remote** — this file, fetched over the network (5-second timeout)
2. **On-device cache** — the last successfully fetched config, stored locally
3. **Hardcoded defaults** — baked into the app binary at build time

If this repository or GitHub itself becomes unreachable, users will continue to see the last config they successfully fetched.
