# sleepa-live

A public, live-updating NFL scoreboard: every game this week, with live play-by-play (phase 2), and any Sleeper
username's players and opponents' players marked on plays (phases 3–4). `index.html` is the whole app. Read
`PROJECT_CONTEXT.md` for how it works and its data sources.

## Security rules (public repo)

Anyone can read this repo. Keep it that way safely:

- **No secrets in files, ever.** The app uses only public, keyless APIs (ESPN site API, Sleeper). If a key is ever
  needed it lives in GitHub repository secrets and reaches scripts through `env:`; scripts never print it.
- **index.html loads no third-party JavaScript** (only Google Fonts CSS). No CDN scripts.
- **Escape everything other people control** before it goes into HTML: play text, team / player names, Sleeper
  usernames, display names, league and team names. Use `escapeHTML()`. Image URLs must be `https://` (`safeImg`).
  Colors from APIs must match `/^[0-9a-f]{6}$/i` (`safeColor`).
- **The Sleeper username stays in localStorage**, never in the URL or sent anywhere except Sleeper's own API.
- **Polling stays polite:** one honest request pattern, pause when the tab is hidden, full game summaries
  (~430 KB) only for the game the viewer has open.
- **Actions (if ever added)** are pinned to full commit SHAs with the version in a comment; triggers stay
  `schedule` / `workflow_dispatch`; `permissions:` as narrow as the job needs; Python is standard library only.
- **Don't rewrite pushed history.** Add files by name rather than `git add -A`.
- Commit as `adambar-io@users.noreply.github.com`, never a personal email.
