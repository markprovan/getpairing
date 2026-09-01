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

Deployed to **Cloudflare Pages** (project `getpairing`, a **Direct Upload**
project — Cloudflare is deliberately *not* connected to this repo). GitHub
Actions is the only thing that builds the site; wrangler just ships the output.

- Push to `master` → `.github/workflows/deploy.yml` builds and publishes to
  production.
- Open a PR → the `preview` job in `ci.yml` deploys the same artifact to a
  preview URL and comments it on the PR. It skips cleanly if the Cloudflare
  secrets aren't set, so a missing secret never blocks a PR.
- `corepack pnpm deploy` does a production deploy locally (needs
  `wrangler login`).

Required repository secrets:

- `CLOUDFLARE_API_TOKEN` — with the *Cloudflare Pages: Edit* permission
- `CLOUDFLARE_ACCOUNT_ID`

Two things that will bite if changed: the Pages project's **production branch
must be `master`** to match the `--branch=master` flag, or uploads land as
preview deployments instead of going live; and a Direct Upload project
[cannot be converted to a Git-connected one](https://developers.cloudflare.com/pages/get-started/direct-upload/)
(or back) — that needs a new project.

Both workflows **fail the build if no font files land in `dist/`**. Astro fetches
the faces at build time and a fetch failure is only a *warning*, so without that
check a build would succeed and silently ship the fallback stacks. If it trips,
look for "No data found for font family" in the build log.

## Theme

The design system is ported from [markprovan.com](https://markprovan.com)
(`markprovan-static`) so the two sites read as siblings: cream/warm-umber
palette, sage accent, terracotta links, and Fraunces / Newsreader / Inter.

- **Tokens** live at the top of `src/styles/global.css`. The dark palette is a
  token override under `:root[data-theme="dark"]` — no separate stylesheet.
- **Dark mode** is `src/components/ThemeToggle.astro`. An `is:inline` script in
  `DocsLayout` sets `data-theme` before first paint so there's no flash, and the
  toggle re-asserts it after view transitions. The choice persists in
  `localStorage`; with nothing stored it follows the OS setting.
- **Fonts are self-hosted at build time** via Astro's Fonts API — no runtime
  requests to Google. Note this means the build needs network access to
  `fonts.google.com`, and a fetch failure is only a *warning*: the build still
  succeeds but silently ships the fallback stacks (Georgia / system sans). If
  the type looks wrong on a deploy, check the build log for
  "No data found for font family".
- **The sidebar nav is explicit** (`src/components/Sidebar.astro`) so ordering
  and labels are deliberate, but a build-time check fails the build if a page
  under `src/pages` isn't linked from it — adding a page and forgetting the nav
  is an error, not a silent omission.
- Below 900px the sidebar collapses into a `<details>` disclosure; above it, it
  sits alongside the content and the summary is hidden.

## Dependency policy

`pnpm-workspace.yaml` sets `minimumReleaseAge: 1440` (one day) and
`.github/dependabot.yml` sets a matching `cooldown`, so a release isn't adopted
the moment it publishes. This is deliberate — see the comments in both files for
the Astro/MDX version skew that motivated it.
