# Sleepa Live: project context

A sibling of Sleepa (adambar-io/sleepa). Same look (tokens copied from Sleepa's index.html), separate repo, no
shared code: pieces are copied over when needed. Hosted on GitHub Pages at adambar-io.github.io/sleepa-live/ —
the same origin as Sleepa, which matters because Sleeper's GraphQL play feed refuses localhost (and maybe other
origins).

## Phases

1. **All-games board** (built Nov 22 2026). A card per game: score, clock, possession, timeouts, field-position bar
   with first-down line and down & distance, last play, ESPN win probability, pass / rush / receiving leaders,
   score by quarter. Live first, then upcoming, then final. Week switcher (only the current week polls).
2. **Game view.** Drive-by-drive play feed, new plays sliding in, scoring summary, win-probability line, box score.
3. **Your players.** Sleeper username → leagues → my starters and each opponent's starters, marked on plays across
   all games, tagged by league.
4. **Fantasy points per play** in each league's scoring, plus a "My plays" feed across games.

## Data sources (checked live Nov 22 2026)

- **ESPN scoreboard** `site.api.espn.com/apis/site/v2/sports/football/nfl/scoreboard` (`?week=N&seasontype=2`
  for another week). CORS `*`, `max-age=1`. Per event: `status` (state pre/in/post, period, displayClock,
  shortDetail, type.name e.g. STATUS_HALFTIME / STATUS_END_PERIOD), competitors (team id, abbreviation, color,
  logo / logoDark, score, linescores, records), `situation` (down, distance, `yardLine`, downDistanceText,
  possession = team id, isRedZone, home/awayTimeouts, `lastPlay` with text, type and
  `probability.homeWinPercentage`), `leaders` (passing / rushing / receiving), broadcasts, `odds` (pregame),
  `weather`, `neutralSite`. `week.teamsOnBye`.
  - **`yardLine` is yards from the HOME team's goal line** (CHI home "at CHI 37" → 37; TEN away "at TEN 11" →
    89). The field bar draws the away end zone on the left and home on the right, ball at `100 − yardLine` %.
- **ESPN game summary** `…/summary?event=<id>`: drives (current / previous) with plays; each play has clock,
  period, score, type, text, `participants` (ESPN athlete id, name, headshot, position, team); also
  winprobability series, scoringPlays, boxscore, leaders. ~430 KB per call → poll only the open game (phase 2).
- **Sleeper** (phase 3–4): `api.sleeper.app/v1` (user, leagues, rosters, matchups) and GraphQL
  `sleeper.com/graphql` `plays` (stats per play; `plays(game_id)` = newest 20 plays, `plays(week)` ≈ 3.4 MB).
  See Sleepa's PROJECT_CONTEXT.md "Source" note and `wpcFetch` / `zPend` in Sleepa's index.html.

## Polling

`nextDelay()`: 10 s while any game is live, 30 s when a kickoff is within 15 minutes, otherwise 2 minutes (capped
to wake at the next kickoff). Paused while the tab is hidden; refetches on return. A failed fetch keeps the last
board and retries. Cards are keyed by event id and only replaced when their HTML changes, so score / new-play
flashes fire only on real changes.
