# AWS Amplify Smoke Test

Smoke test run against:

```text
https://gsc.athbheochan.irish
```

Run date: 2026-08-15

## Result

Partial pass. Static hosting, HTTPS, routes, redirects, and static assets are
working. Supabase-backed frontend behavior is blocked because the deployed
browser bundle does not appear to include the public Supabase project URL.

## Passed

- `https://gsc.athbheochan.irish/` returns `200` over HTTPS.
- `https://gsc.athbheochan.irish/status.json` returns `200` and valid JSON.
- `https://gsc.athbheochan.irish/logo.svg` returns `200`.
- `https://gsc.athbheochan.irish/covers/spyro1.png` returns `200`.
- `https://gsc.athbheochan.irish/guide/` returns `200`.
- `https://gsc.athbheochan.irish/contribute/` returns `200`.
- `https://gsc.athbheochan.irish/games/ps1/spyro1/` returns `200`.
- `https://gsc.athbheochan.irish/games/ps1/spyro1/strings/` returns `200`.
- `https://gsc.athbheochan.irish/games/ps1/spyro1/notes/` returns `200`.
- No-slash page URLs redirect to slash routes:
  - `/guide` -> `/guide/`
  - `/games/ps1/spyro1` -> `/games/ps1/spyro1/`
  - `/games/ps1/spyro1/strings` -> `/games/ps1/spyro1/strings/`
- Deployed client chunks do not contain private Supabase markers:
  - `SUPABASE_SERVICE_KEY`
  - `SUPABASE_ANON_KEY`
  - `service_role`
  - `service-role`

## Blocked

Supabase-backed reads and contribution flows are not confirmed yet. The deployed
client chunks did not contain the expected public Supabase project URL:

```text
https://csghgkqinmxhclenaair.supabase.co
```

The deployed `/_app/env.js` response was:

```js
export const env={}
```

Because the site code uses build-time public Supabase constants, this likely
means the current Amplify deployment was built without these environment
variables:

```text
PUBLIC_SUPABASE_URL
PUBLIC_SUPABASE_PUBLISHABLE_KEY
```

Set those values in Amplify, redeploy `main`, then rerun the Supabase portions
of this smoke test.

## Not Fully Verified

- Browser console runtime errors were not fully verified because no working
  browser automation dependency is available in the repo.
- Mobile viewport visual testing was attempted with headless Firefox, but the
  process hung before producing a screenshot.
- Suggestion, vote, contributor, and GitHub OAuth flows were not exercised
  because Supabase-backed client configuration appears absent from the deployed
  bundle.

## Rerun Commands

```bash
curl -I https://gsc.athbheochan.irish/
curl -I https://gsc.athbheochan.irish/status.json
curl -I https://gsc.athbheochan.irish/logo.svg
curl -I https://gsc.athbheochan.irish/covers/spyro1.png
curl -I https://gsc.athbheochan.irish/guide/
curl -I https://gsc.athbheochan.irish/contribute/
curl -I https://gsc.athbheochan.irish/games/ps1/spyro1/
curl -I https://gsc.athbheochan.irish/games/ps1/spyro1/strings/
curl -I https://gsc.athbheochan.irish/games/ps1/spyro1/notes/
curl -L https://gsc.athbheochan.irish/status.json | python3 -m json.tool >/tmp/gsc-status-json-check.txt
```

To rescan deployed browser-visible JS for Supabase configuration, download the
client chunks referenced by the strings page and scan them for:

```text
csghgkqinmxhclenaair
SUPABASE_SERVICE_KEY
SUPABASE_ANON_KEY
service_role
service-role
```
