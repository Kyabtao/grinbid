# 🧪 Grinbid — static demo

A **fully working, 100% static demo** of the Grinbid app. Every screen, every
button and every game mechanic works — but there is **no server**: a mock
backend runs inside the page, demo data lives in your browser's
`localStorage`, and the "live" feed is simulated by demo bots.

```
demo/
  index.html     the page — just open it (no build step, no install)
  styles.css     playful-pop theme + demo chrome + mobile responsive layout
                 (hamburger menu, stacked grids, bottom-docked toasts)
  demo-data.js   static demo seed: 13 fan pages, 16 tasks, 7 demo users,
                 33-boost history, claim queue, donation intents
  demo-api.js    the in-browser mock backend: same routes & economy math as
                 the real API (streaks, drops, ×1.5 self-boosts, 2s cooldown,
                 task unlocks, referral match, season payouts, admin tools)
  app.js         the SPA: hash routing, 9 screens, modals, confetti,
                 live-feed toasts, cross-tab sync — re-renders are
                 non-destructive (no full-page flash, scroll is kept,
                 and live updates never interrupt you while typing)
  server.js      optional zero-dependency static server (not required)
  test-demo.js   headless check of the mock API — `node demo/test-demo.js`
                 (29 assertions: economy math, cooldowns, tasks, referrals,
                 admin flows, season payouts, reseed)
  check-wiring.js static check that every UI handler/API call resolves —
                 `node demo/check-wiring.js`
```

## Run it

```bash
# option 1 — just open it
open demo/index.html            # macOS    (double-click works too)

# option 2 — tiny static server (optional)
node demo/server.js             # → http://localhost:4173

# option 3 — anything that serves files
python3 -m http.server 4173 --directory demo
```

No `npm install`, no environment variables, no network calls.

## Deploy on GitHub Pages

Two one-time steps and every push to `main` that touches `demo/` goes live
automatically at **`https://<owner>.github.io/grinbid/`**:

1. **Add the workflow** — on GitHub: *Add file → Create new file*, name it
   `.github/workflows/deploy-demo.yml`, paste the contents of
   [`demo/github-pages-workflow.yml`](github-pages-workflow.yml), commit to `main`.
2. **Enable Pages** — *Settings → Pages → Build and deployment → Source:
   GitHub Actions*.

Then merge the demo PR (or push any change inside `demo/`) and watch the run
under the *Actions* tab. You can also trigger it manually from
*Actions → Deploy demo to GitHub Pages → Run workflow*.

Works because the demo is 100% static: relative asset paths, hash routing,
no server calls — only `localStorage`.

## Demo logins

| Account | Password | What it shows |
|---|---|---|
| `demo_fan` | `demo1234` | 8,730 coins · day-4 streak (claim → day 5!) · ready lucky drop · claimable tasks · 2 referrals |
| `pixelpanda` | `demo1234` | owns the *Midnight Mosaic* fan page → try the **×1.5 self-boost** |
| `moonwalker_z` | `demo1234` | top of the season leaderboard |
| Admin console | `grinbid-admin-dev` | stats, broadcast, notify, season settle, claim approve/reject, reseed |

The login modal has one-click buttons for all of these.

## What works (everything!)

- **Auth** — sign up (+2,500 coin bonus, avatar picker, referral code), log in,
  log out, sessions. New signups with `GB-DEMO42-A1B2` (demo_fan's code) earn
  the referee/referrer bonuses + 10% lifetime match.
- **Discover** — search, category filter, boost meters, fan-made / community /
  🟢 verified badges.
- **Profiles** — boost (min 50, 2-second cooldown, ×1.5 on your own page),
  recent-boost history, fan counts, live ranking.
- **Wallet** — daily streak claim (500 + 150/day, capped 2,000), 3-hour lucky
  drop with live countdown (250–2,500 coins), full transaction history.
- **Tasks** — all 16 tasks with staged unlocks, auto-completion and claimable
  rewards (cascading unlocks included).
- **Referrals** — invite code, share, squad list with pending-review flags,
  lifetime match earnings.
- **Create** — one community page per user, then self-boost it at ×1.5.
- **Donate** — strictly non-reward intents (no coins, ever).
- **Admin** — overview stats, SSE-style broadcast, user notify, forced season
  settlement (50k/25k/10k prizes to the top 3), claim-request approve/reject,
  full demo reset.
- **Live feel** — demo bots boost around every ~10s, presence counter moves,
  and open tabs stay in sync with each other. Live updates repaint the screen
  in place: no reload-style flash, your scroll position and any text you're
  typing stay untouched.
- **Mobile responsive** — hamburger menu on narrow screens, stacked grids,
  full-width buttons, bottom-docked toasts, no sideways scrolling.

## Reset

Use the **♻️ Reset demo data** button in the yellow strip at the top of the
page (or Admin → reseed). Demo data is stored only in your browser — clearing
the site's localStorage removes it completely.

## Honest small print

This demo fakes persistence (localStorage instead of the atomic JSON store),
sessions (no scrypt/HMAC cookies), rate limiting, SSE (an in-page event bus +
BroadcastChannel) and payments (nothing is ever charged anywhere). The coin
economy math itself mirrors `src/economy.js` faithfully. Coins are and always
will be 100% free virtual coins with zero cash value.
