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

Not wired up yet. The site is destined for **Cloudflare Pages**, but the CI
deploy job is deliberately absent until the `getpairing` Pages project and its
credentials exist — a workflow that fails on every push to `master` is worse
than no workflow.

`corepack pnpm deploy` works locally today if you have `wrangler` authenticated
(`wrangler login`). Restoring the automated deploy needs:

- repo secrets `CLOUDFLARE_API_TOKEN` (with *Cloudflare Pages: Edit*) and
  `CLOUDFLARE_ACCOUNT_ID`
- a `getpairing` Pages project whose **production branch is `master`**, to match
  the `--branch=master` flag — otherwise uploads land as preview deployments
  instead of going live
- DNS for getpairing.com pointed at Pages, and the old Netlify site retired

## Dependency policy

`pnpm-workspace.yaml` sets `minimumReleaseAge: 1440` (one day) and
`.github/dependabot.yml` sets a matching `cooldown`, so a release isn't adopted
the moment it publishes. This is deliberate — see the comments in both files for
the Astro/MDX version skew that motivated it.
