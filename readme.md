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
