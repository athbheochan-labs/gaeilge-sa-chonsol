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

## Provisioning Checklist

1. Create an Amplify app from the GitHub repository.
2. Connect the `main` branch.
3. Confirm Amplify detects and uses `amplify.yml`.
4. Add `PUBLIC_SUPABASE_URL` and `PUBLIC_SUPABASE_PUBLISHABLE_KEY` in Amplify.
5. Leave `BASE_PATH` unset.
6. Add the custom domain `gsc.athbheochan.irish` in Amplify.
7. Deploy the branch and record both the Amplify HTTPS URL and production domain.
8. Smoke test the deployed site with issue #169.
