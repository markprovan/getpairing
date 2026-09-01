# Get Pairing

A guide to get you started, or improve your pair programming experience.

Built with [Astro](https://astro.build/) and TypeScript.

## Development

Requires Node 22 and pnpm (pinned via `packageManager` — use `corepack pnpm …`).

```sh
corepack pnpm install
corepack pnpm dev
```

## Build

```sh
corepack pnpm build      # static output in dist/
corepack pnpm typecheck  # tsc --noEmit
corepack pnpm preview    # serve the built site locally
```

## Deployment

Pushing to `master` runs `.github/workflows/deploy.yml`, which builds and
publishes `dist/` to **Cloudflare Pages** (project `getpairing`). It needs two
repository secrets:

- `CLOUDFLARE_API_TOKEN` — with the *Cloudflare Pages: Edit* permission
- `CLOUDFLARE_ACCOUNT_ID`

The Pages project's **production branch must be `master`** to match the
`--branch=master` flag, otherwise uploads land as preview deployments instead of
going live. `corepack pnpm deploy` does the same thing locally as an escape hatch.

## Dependency policy

`pnpm-workspace.yaml` sets `minimumReleaseAge: 1440` (one day) and
`.github/dependabot.yml` sets a matching `cooldown`, so a release isn't adopted
the moment it publishes. This is deliberate — see the comments in both files for
the Astro/MDX version skew that motivated it.
