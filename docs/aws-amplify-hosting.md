# AWS Amplify Hosting

This project is hosted on AWS with Amplify Hosting. The site remains a static
SvelteKit build and Supabase remains the runtime data service for contribution
features.

## Decision

Use Amplify Hosting rather than S3 + CloudFront for the initial AWS migration.
The project does not need Cognito, Lambda, API Gateway, custom CloudFront
behaviors, or a custom domain at this stage. Amplify gives the project an
AWS-managed HTTPS preview URL, build logs, branch deploys, and environment
variable management with less infrastructure to maintain.

## AWS Resources

- Amplify app connected to the GitHub repository.
- Amplify branch for `main`.
- Amplify-managed HTTPS hosting URL.
- Custom domain: `https://gsc.athbheochan.irish`.

Not required for the initial migration:

- S3 bucket or CloudFront distribution managed directly by this repository.
- Cognito or AWS-hosted authentication.
- Static fallback rewrites, because the SvelteKit build prerenders route
  directories with `index.html` files.

## Build Settings

Amplify should use the repository-root [`amplify.yml`](../amplify.yml) file.

Build summary:

- App root: `web`
- Node runtime: Node 20 via `nvm install 20` and `nvm use 20`
- Python runtime: Python 3.11 expected in the Amplify build image
- Install command: `npm ci`
- Build command: `npm run build`
- Build output: `web/build`
- `BASE_PATH`: leave unset for AWS root hosting

Amplify owns the production build/deploy pipeline. Pushes to the connected
`main` branch should trigger an Amplify build from `web/` and publish the
resulting static files from `web/build`.

Required Amplify environment variables:

- `AMPLIFY_MONOREPO_APP_ROOT=web`
- `PUBLIC_SUPABASE_URL`
- `PUBLIC_SUPABASE_PUBLISHABLE_KEY`

Do not add Supabase service-role secrets to Amplify for the static frontend.
Server/admin Supabase keys remain for CI or local maintenance workflows only.

No GitHub Actions AWS secrets or OIDC role are required for the Amplify-native
pipeline. The AWS-side integration is the Amplify GitHub App connection, which
needs access to this repository so Amplify can create webhooks and receive push
events.

The legacy GitHub Pages workflow at
`.github/workflows/deploy-pages.yml` is manual-only and is superseded by
Amplify for production deploys.

## Served Files

- Static SvelteKit pages are emitted as directory routes, such as
  `/guide/`, `/contribute/`, and `/games/ps1/spyro1/`.
- `status.json` is served from `/status.json`.
- Cover images are served from `/covers/`.
- The site logo is served from `/logo.svg`.
- Generated game notes are served from prerendered notes routes, such as
  `/games/ps1/spyro1/notes/`.
- Release links continue to point to GitHub Releases unless release artifact
  hosting is changed in a separate migration ticket.

## Domain

The production site URL is:

```text
https://gsc.athbheochan.irish
```

Keep `BASE_PATH` unset for this domain. The site is served from the domain root,
so app routes and static files resolve as `/guide/`, `/status.json`,
`/covers/spyro1.png`, and similar root-relative paths.

Supabase Auth settings should also allow this domain when configuring the
AWS-hosted site:

```text
https://gsc.athbheochan.irish/**
```

## Supabase Environment

The Amplify-hosted frontend keeps using the existing Supabase project. No
Supabase schema, RLS policy, or private backend change is required for the AWS
hosting migration.

Configure Amplify with only browser-safe Supabase values:

```text
PUBLIC_SUPABASE_URL=https://csghgkqinmxhclenaair.supabase.co
PUBLIC_SUPABASE_PUBLISHABLE_KEY=<Supabase publishable key>
```

Do not configure these in Amplify:

```text
SUPABASE_SERVICE_KEY
SUPABASE_ANON_KEY
SUPABASE_URL
```

The app only reads `PUBLIC_SUPABASE_URL` and
`PUBLIC_SUPABASE_PUBLISHABLE_KEY` during the frontend build. Admin/service-role
keys are reserved for local scripts and GitHub Actions maintenance workflows
that sync or seed Supabase data.

In Supabase Auth URL configuration:

- Set the production site URL to `https://gsc.athbheochan.irish`.
- Add `https://gsc.athbheochan.irish/**` to allowed redirect URLs.
- Add the Amplify branch URL, such as
  `https://main.<amplify-app-id>.amplifyapp.com/**`, while smoke testing the
  AWS preview URL.
- Keep localhost redirect URLs only if local OAuth testing is still needed.

If GitHub OAuth is enabled in Supabase, the GitHub OAuth app callback remains
the Supabase callback URL. The AWS site URL belongs in Supabase redirect
settings, not in the GitHub OAuth callback directly.

After deployment, verify the strings page can:

- Read public string rows from Supabase.
- Load suggestions and vote counts.
- Sign in through GitHub OAuth and return to the same Amplify/custom-domain
  route.
- Submit suggestions.
- Vote on suggestions.
- Check contributor status before verification actions.

To confirm private keys are not shipped, build the site and scan the static
output for private Supabase markers:

```bash
cd web
npm run build
rg "SUPABASE_SERVICE_KEY|service_role|service-role|SUPABASE_ANON_KEY" build .svelte-kit/output/client
```

The scan should return no matches.

## Provisioning Checklist

1. Create an Amplify app from the GitHub repository.
2. Connect the `main` branch.
3. Confirm Amplify detects and uses `amplify.yml`.
4. Add `PUBLIC_SUPABASE_URL` and `PUBLIC_SUPABASE_PUBLISHABLE_KEY` in Amplify.
5. Leave `BASE_PATH` unset.
6. Add the custom domain `gsc.athbheochan.irish` in Amplify.
7. Deploy the branch and record both the Amplify HTTPS URL and production domain.
8. Smoke test the deployed site with issue #169.
