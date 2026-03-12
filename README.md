# Slice Hibernation Config

This repository is the remote configuration source for the Slice mobile app during its hibernation period.

After Day 0 (when backend infrastructure is decommissioned), this repo is the **only mechanism** for changing app behaviour without an App Store release. The app fetches `hibernation-config.json` on every launch. If the fetch succeeds, values are stored on-device and used. If the fetch fails, the app falls back to the last cached values, then to hardcoded defaults.

## How to make a change

1. Edit `hibernation-config.json` directly in the GitHub web UI (no local setup needed)
2. Commit to `main`
3. Users will pick up the change on their **next app open** (subject to a 5-second network timeout)

---

## Config reference

The config root has three keys — `dev`, `staging`, and `production` — one per environment. The app uses the object for the current build (e.g. production builds use `production`). Each object has the same shape:

```json
{
  "dev": {
    "hibernationStartAt": "2026-03-09T00:00:00.000Z",
    "portOutDeadlineAt": "2026-05-31T00:00:00.000Z",
    "closureDate": "31 May 2026",
    "portOutDeadline": "31 May 2026",
    "links": {
      "home": "https://www.slicemobile.com",
      "terms": "https://www.slicemobile.com/terms-conditions",
      "privacy": "https://www.slicemobile.com/privacy-policy",
      "closureFaq": "https://help.slicemobile.com/en/",
      "dataRequest": "https://www.slicemobile.com/privacy-policy"
    }
  },
  "staging": { ... },
  "production": { ... }
}
```

| Field                | Type         | Description                                                                                                                                                  |
| -------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `hibernationStartAt` | ISO datetime | When the 3-screen warning flow activates (closing notice → port your number → need help). Set in advance; the app self-activates when the clock passes this. |
| `portOutDeadlineAt`  | ISO datetime | When the final "IT'S BEEN SLICE." screen activates (port-out deadline has passed, service fully closed).                                                     |
| `closureDate`        | string       | Display string for the service end date, shown in-app (e.g. `"31 May 2026"`).                                                                                |
| `portOutDeadline`    | string       | Display string for the port-out deadline, shown in-app (e.g. `"31 May 2026"`).                                                                               |
| `links.home`         | string       | Main website URL. Update when `slicemobile.com` goes offline.                                                                                                |
| `links.terms`        | string       | Terms & Conditions URL.                                                                                                                                      |
| `links.privacy`      | string       | Privacy Policy URL.                                                                                                                                          |
| `links.closureFaq`   | string       | FAQ page for the closure. Update when the help site goes offline.                                                                                            |
| `links.dataRequest`  | string       | URL for users to submit a data request.                                                                                                                      |

---

## App flow

```
now < hibernationStartAt
  → Normal app

hibernationStartAt <= now < portOutDeadlineAt
  → 3-screen warning flow (Slice is closing → Port your number → Need help)

now >= portOutDeadlineAt
  → Final screen: "IT'S BEEN SLICE." (full sleep, no navigation)
```

---

## Common scenarios

**Activating the warning flow:**
Set `hibernationStartAt` to the intended datetime in ISO format (e.g. `"2026-04-01T08:00:00.000Z"`). Commit now — the app self-activates at the right moment without a midnight push.

**Activating full sleep (service closed):**
Set `portOutDeadlineAt` to the port-out deadline datetime. The final screen appears automatically once that time passes.

**Testing before going live:**
Use different payloads per environment: e.g. set `dev` and `staging` with past dates so the flow is active there, and set `production` with far-future dates so production users don't see the flow until you're ready.

**The closure or deadline dates change:**
Update `closureDate`, `portOutDeadline` (display strings) and `portOutDeadlineAt` (machine datetime) accordingly and commit.

**`slicemobile.com` goes offline:**
Update all affected `links.*` fields to point at alternative URLs (e.g. GitHub Pages at `paraspara1.github.io/...`) and commit.

**Funding is secured:**
Set both `hibernationStartAt` and `portOutDeadlineAt` to a far-future date (e.g. `"2099-01-01T00:00:00.000Z"`). A full App Store update will still be needed to restore normal functionality.

---

## Fallback behaviour

The app resolves config in this order on every launch:

1. **Remote** — this file, fetched over the network (5-second timeout)
2. **On-device cache** — the last successfully fetched config, stored locally
3. **Hardcoded defaults** — baked into the app binary at build time

If this repository or GitHub itself becomes unreachable, users will continue to see the last config they successfully fetched.
