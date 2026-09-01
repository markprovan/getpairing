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

Deployed to **Cloudflare Workers** (static assets), not Pages. Cloudflare now
[recommends Workers for new projects](https://developers.cloudflare.com/workers/static-assets/migration-guides/migrate-from-pages/)
and wrangler nudges you off Pages, so that's where this lives.

`wrangler.jsonc` declares an **assets-only Worker**: there's no `main`, so no
Worker script runs at all — Cloudflare serves `dist/` from its asset store.
`not_found_handling: "404-page"` makes unmatched paths serve our built
`404.astro` instead of a bare Cloudflare error.

GitHub Actions is the only thing that builds the site; wrangler just ships the
output. Cloudflare is deliberately *not* connected to the repo, so there's one
build environment rather than two places for build config to drift.

- Push to `master` → `deploy.yml` builds and runs `wrangler deploy`.
- Open a PR → the `preview` job runs `wrangler versions upload`, which uploads a
  Worker version **without** promoting it to production and returns a preview URL,
  posted as a PR comment that updates in place. Production only ever changes via
  `wrangler deploy`.
- `corepack pnpm deploy` deploys to production locally (needs `wrangler login`).

Required repository secrets:

- `CLOUDFLARE_API_TOKEN` — with permission to edit Workers
- `CLOUDFLARE_ACCOUNT_ID`

**The `name` in `wrangler.jsonc` must match the Worker in the dashboard.**
Unlike Pages — which errors when the project is missing — `wrangler deploy`
*creates* a Worker when the name doesn't exist. A typo therefore produces a
second, silently-working Worker while the custom domain stays pointed at the
original.

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
