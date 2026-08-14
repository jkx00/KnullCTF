# KNULL CTF theme

A dark, emerald-accented "security operations center" theme for [CTFd](https://ctfd.io),
built on top of `core-beta` (Bootstrap 5 + Alpine.js + Vite).

This is a **visual theme only**. It does not touch CTFd's backend, database,
authentication, challenge engine, or admin panel — it restyles the
participant-facing templates that CTFd already ships and reuses CTFd's
existing Jinja context, JS API client (`@ctfdio/ctfd-js`), and Alpine
components wherever possible.

## What changed vs. `core-beta`

- **Layout**: sticky top bar + collapsible/off-canvas left sidebar + footer,
  replacing the single Bootstrap navbar (`templates/base.html`,
  `templates/components/topbar.html`, `templates/components/sidebar.html`).
- **Design system**: a full SCSS token set and component layer
  (`assets/scss/_variables.scss` through `_responsive.scss`) implementing the
  near-black / emerald palette, Inter + JetBrains Mono typography, 16/12/10px
  radius scale, and card/chip/button system described in the brief.
- **Hand-restyled templates**: `challenges.html`, `challenge.html` (the
  challenge modal), `scoreboard.html`, `login.html`, `register.html`,
  `reset_password.html`, `teams/public.html`, `users/public.html`,
  `page.html` (including a real-data dashboard on the homepage), and all
  `errors/*.html` pages.
- **Everything else** (settings, teams/users management pages, notifications,
  confirm, join/new team) is left structurally untouched — it inherits the
  KNULL look for free because those pages use plain Bootstrap classes
  (`.jumbotron`, `.table`, `.card`, `.modal`, `.nav-pills`, `.alert`, …) that
  are globally reskinned in `_bootstrap-overrides.scss` and
  `includes/components/_jumbotron.scss`.
- **New JS**: `assets/js/navigation.js` (sidebar collapse/mobile drawer,
  topbar clock, sidebar event-status countdown) and
  `assets/js/command-palette.js` (Ctrl/Cmd+K search over the same
  already-permission-filtered nav links + `/api/v1/challenges` data), both
  registered once in `index.js` so they're available on every page. A new
  `assets/js/home.js` entry powers the homepage dashboard using
  `/api/v1/users/me` (or `/teams/me`) + `/me/solves` + `/me/awards` — all
  real, already-existing CTFd endpoints.
- Every other CTFd template, JS hook (`CTFd.pages.*`, Alpine `x-data`
  components, Bootstrap `data-bs-*` wiring), block name used by
  challenge-type plugins, and form/CSRF flow was preserved byte-for-byte
  where the file wasn't touched, or preserved attribute-for-attribute where
  it was restyled (see "Preservation notes" below).

## Scope decisions

A few things in the original brief describe data CTFd doesn't actually track,
so rather than inventing fake data these were adapted honestly:

- **No fabricated "first blood" flag on challenge cards.** CTFd's
  `/api/v1/challenges` listing doesn't return a first-blood indicator, and
  computing it per-card would mean one extra request per challenge. The CSS
  (`.chal-card--first-blood`) exists for you to wire up if you add a
  first-blood plugin later.
- **No fabricated "difficulty" badge.** Core CTFd challenges don't have a
  difficulty field; only `category`, `value`, and `tags` are real.
- **Dashboard KPI #4 is "Categories" instead of "First Bloods"** — derived
  from your own real solve history (`/api/v1/users/me/solves`).
- **Scoreboard podium/table show rank, name, bracket, and score only** —
  `/api/v1/scoreboard` doesn't expose country or solve counts, so those
  columns from the brief's mockup were dropped rather than faked.
- **The light/dark toggle was removed.** This theme is intentionally a
  single, deliberate dark palette (`color_mode_switcher.js` was removed);
  nothing server-side depends on it.
- **Sidebar nav only links to real routes.** "Achievements" / "Events" from
  the brief's suggested nav aren't backed by CTFd routes, so they were left
  out rather than linking to a page that doesn't exist. Everything else
  (Overview, Challenges, Scoreboard, Team, Activity/Notifications, Users,
  Teams, Profile, Settings, Admin Panel) maps to a real CTFd endpoint.

## Preservation notes (for reviewers)

- `challenge.html` keeps every id/class that CTFd's JS or other challenge
  type plugins depend on: `#challenge-id`, `#challenge-input`,
  `#challenge-submit`, `.challenge-desc`, `.challenge-connection-info`,
  `.challenge-hints`, `.challenge-files`, the `#challenge` / `#submissions`
  / `#solves` / `#solution` tab panes, and every `{% block %}` name
  (`tags`, `attribution`, `ratings`, `description`, `connection_info`,
  `input`, `submit`, `submissions`, `solution`, `solves`) that
  `/plugins/challenges/assets/view.html` and third-party challenge type
  plugins extend.
- `challenges.html` keeps the exact `x-data="ChallengeBoard"` /
  `@load-challenges.window` / `@load-challenge.window` wiring; the
  search/category filter is additive state on the same Alpine component
  (`assets/js/challenges.js`), not a rewrite of its data flow.
- `scoreboard.html`, `teams/public.html`, `users/public.html` keep their
  `x-data` components, `window.TEAM` / `window.USER` globals, and
  `Assets.js(...)` calls untouched.
- CSRF (`form.nonce()`), WTForms field rendering, and `Forms.*` construction
  are unchanged everywhere.

## Installing

1. Copy (or symlink) this directory into your CTFd install as
   `CTFd/themes/knull` — if you're already inside a CTFd checkout, it's
   already there.
2. In the admin panel, go to **Config → Theme** and select `knull`.
3. The theme ships pre-built (`static/` is committed, same convention as
   `core-beta`), so no build step is required to use it as-is.

## Developing

```bash
cd CTFd/themes/knull
npm install
npm run dev     # vite build --watch, for local development
```

## Building for production

```bash
cd CTFd/themes/knull
npm install
npm run build    # one-time production build into static/
```

`vite.config.js` copies the self-hosted Inter/JetBrains Mono/Font Awesome
webfonts and `assets/img` + `assets/sounds` into `static/` as part of the
build — nothing is loaded from a third-party CDN.

## Verification performed

- `npm run build` completes cleanly (Vite + Sass, Bootstrap 5.3 compiled with
  the KNULL palette via `@use ... with (...)`, no console errors).
- Every template under `templates/` was parsed with `jinja2.Environment.parse()`
  to confirm syntactic correctness (balanced blocks, valid expressions).
- Every `url_for(...)` endpoint referenced in edited templates was
  cross-checked against CTFd's blueprints (`auth`, `challenges`, `scoreboard`,
  `teams`, `users`, `views`, `admin`).
- Every field referenced from CTFd's JSON APIs (challenge listing/detail,
  scoreboard, users/me, teams/me, solves) was cross-checked against the
  corresponding marshmallow schema / API view in `CTFd/api/v1` and
  `CTFd/schemas` rather than assumed.
- **A full end-to-end pass against a real, running CTFd dev server was
  completed** (system Python is 3.14, which the pinned CTFd dependency set
  doesn't install cleanly on; a Python 3.11 interpreter was installed
  side-by-side via [`uv`](https://docs.astral.sh/uv/) — no `sudo`, no change
  to the system interpreter — specifically to unblock this). The instance was
  driven through setup, login, team creation, challenge solving (correct +
  incorrect flag submission, hint unlock), scoreboard, and the homepage
  dashboard, in a real headless browser (Playwright), with console/network
  errors captured at every step. This found and fixed three real bugs, all
  already corrected in this checkout:
  - **CTFd-branded default homepage content.** `CTFd/views.py`'s setup
    handler seeded new installs with a hardcoded CTFd logo image, "A cool CTF
    platform from ctfd.io" copy, and links to `twitter.com/ctfdio` /
    `facebook.com/ctfdio` / `github.com/ctfd`. This bled unstyled CTFd
    branding into the new theme on a fresh install. Rewritten to a neutral
    welcome block that only references the CTF's configured name and, if the
    admin uploads one, their own banner image — no hardcoded third-party
    branding.
  - **Theme shipped CTFd-branded image assets.** `assets/img/favicon.ico` and
    `logo.png` were the literal upstream CTFd wordmark/icon. Regenerated as a
    KNULL "K" mark (emerald favicon) and KNULL CTF wordmark. Five other
    unused CTFd-branded images (`ctfd.ai`, `ctfd.svg`, `ctfd_transfer.svg`,
    `logo_old.png`, `scoreboard.png`) were confirmed unreferenced via grep
    and deleted (~1.4MB of dead weight).
  - **`home.js` fired doomed API calls for teamless users.** In `teams`
    mode, a user who hasn't joined or created a team yet has no team-scoped
    data — the dashboard was still requesting `/api/v1/teams/me` and related
    endpoints, guaranteed to 403. Added an early-return guard so it renders
    the empty state directly instead.
  - Also confirmed, and left as-is, two things that looked like bugs on
    first glance but are correct CTFd behavior: a teamless admin getting 403
    on `/events` and `/api/v1/teams/me*` (core team-gating, already handled
    gracefully by this theme's existing try/catch + `|| []` fallbacks), and
    `POST /api/v1/teams` not auto-joining the creator (that endpoint is
    admin CRUD for team records, not a self-service "create my team" action
    — the real self-join flow is the `/teams/new` form, which works as
    expected).
- The literal string "CTFd" was swept for and replaced with "KNULL CTF" or
  removed everywhere it was purely cosmetic/theme-facing (footer, page
  titles, the setup wizard's section copy, the homepage seed content above).
  Occurrences of "CTFd" that are the actual software's name — the PyPI/npm
  package name, module and config-key names, the MajorLeagueCyber/CTFd
  developer attribution, and the CTFd LLC newsletter opt-in text on the setup
  page — were intentionally left alone as accurate, out of scope for a theme,
  and (for the attribution/newsletter text) not honest to remove.

## Acceptance checklist

- [x] Login / Register / Logout
- [x] Challenge board loads, search + category filters work
- [x] Challenge modal opens, hints unlock, flags submit (correct +
      incorrect), solved state updates live
- [x] Scoreboard loads
- [x] Team creation flow (`/teams/new`) works
- [x] Homepage dashboard renders real score/solves/category data for an
      authenticated user with a team, and a clean empty state for one
      without
- [x] CSRF still functional on every form exercised above
- [ ] File-download-bearing challenges, bracket filter on scoreboard, team
      invite/roster flows, user public/private pages, settings (profile +
      access tokens) — not yet exercised against a live instance; the
      templates/JS were preserved byte-for-byte or attribute-for-attribute
      (see "Preservation notes") but give these a manual pass before
      shipping
- [ ] Custom/plugin challenge types still render (they extend
      `templates/challenge.html`'s blocks) — verified by inspection only,
      no third-party plugin was available to test live
- [ ] Mobile layout: sidebar becomes an off-canvas drawer, tables scroll
      horizontally instead of breaking layout
- [x] Admin Panel link appears only for admins and opens `/admin`
- [x] No flags or hidden challenge data present in page source

## Notes for local dev/testing

If you hit dependency install failures on CTFd's pinned `requirements.txt`
with a very new Python (e.g. 3.13/3.14 removed APIs the pins rely on, such as
`ast.Str`), it's a CTFd/environment issue, not a theme issue. Installing a
3.11 interpreter side by side with [`uv`](https://docs.astral.sh/uv/)
(`uv python install 3.11 && uv venv --python 3.11`) and installing CTFd's
`requirements.txt` into that venv resolves it without touching your system
Python. Also note CTFd caches config reads (`get_config()` /
`Flask-Caching`, filesystem backend by default at `.data/filesystem_cache/`
when no `REDIS_HOST` is set) — if you reset `ctfd.db` to re-run setup, delete
that cache directory too, or stale config (including whether setup has run)
will linger.
