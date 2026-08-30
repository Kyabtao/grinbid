# 🎪 Grinbid — Bid. Back. Rank up.

A playful, **100% free** virtual-coin fan-boost game. Boost faves (celebrities,
estates, venues, brands, communities — all fan-made pages), climb the season
leaderboard, and never spend a single real cent.

```
┌──────────────────────────────────────────────────────┐
│  Zero npm dependencies  ·  Node built-ins only        │
│  Single-file JSON store (data/db.json, atomic rename) │
│  Real-time via Server-Sent Events (/api/stream)       │
│  Vanilla JS + CSS hash-routed SPA (/public)           │
└──────────────────────────────────────────────────────┘
```

---

## Run it

```bash
node server.js          # or: npm start
# → http://localhost:3000
```

Requirements: Node.js ≥ 18 (uses `crypto.scrypt`, `fetch`, `node:test`).
**There is no `npm install` step — the repo has zero dependencies.**

Useful scripts:

| Command | What it does |
|---|---|
| `npm start` | Run the server |
| `npm test` | Full unit + HTTP integration suite (`node --test`) |
| `npm run check` | Zero-dependency + syntax audit on every backend/frontend JS file |
| `npm run audit` | `check` + `test` |
| `npm run audit:loop` | The 20-consecutive-pass audit loop (resets streak on any failure) |
| `npm run reset` | Wipe `data/db.json` and reseed on next boot |

---

## Game economy (the numbers)

| Mechanic | Value |
|---|---|
| Signup bonus | 2,500 coins |
| Daily claim | 500 base + 150/streak day, **capped at 2,000** |
| Lucky drop | 250–2,500 coins every 3 hours |
| Referrals | referrer +1,000 · referee +500 · **10% lifetime match** on referee earnings |
| Season prizes | 50,000 / 25,000 / 10,000 coins (top 3 by season points, auto-settled weekly) |
| Boost | min 50 coins · 1× points normally · **1.5× on your own fan page** · 2 s cooldown |
| Task checklist | 16 tasks with staged unlocks, rewards +100 → +2,000 |

All values live in **`src/config.js`** and all math lives in **`src/economy.js`**
(pure, unit-tested functions). Coins are virtual, free and non-redeemable.

---

## Project layout

```
server.js               bootstrap → src/app.js
src/
  app.js                store + SSE + router + HTTP server assembly
  config.js             every tunable (economy, rate limits, auth, store)
  store.js              JsonStore — atomic rename, fsync, debounced flush,
                        corrupt-file backup & reseed
  seed.js               16 tasks + 12 seeded fan-made profiles
  economy.js            coin math, streaks, drops, boosts, referrals, tasks, seasons
  api.js                HTTP handlers + router wiring (auth, profiles, admin…)
  auth.js               scrypt password hashing + HMAC session cookies
  rateLimit.js          per-IP token buckets
  sanitize.js           input sanitization (length caps, control chars, emoji)
  router.js             tiny framework-free router + static serving
  sse.js                server-sent events hub
public/
  index.html            SPA shell
  app.js                hash-routed SPA: 9 screens + legal pages + SSE client
  styles.css            playful-pop theme (rounded cards, chunky borders,
                        sticker badges, emoji avatars, confetti)
  terms.html · privacy.html   legal pages (also served at /terms, /privacy)
test/
  economy.test.js       daily/streak/drop/boost/referral/task/season math
  store.test.js         atomic persistence, debounce, corrupt recovery,
                        rate limiter, SSE hub
  http.test.js          end-to-end HTTP: auth, boosts, claims, admin, SSE…
scripts/
  check-syntax.js       zero-dep + syntax audit
  audit-loop.js         20-pass audit loop
  reset-data.js         wipe data
```

---

## SPA screens (hash routes)

`#/home` · `#/discover` · `#/profile/:slug` · `#/wallet` · `#/tasks` ·
`#/refer` · `#/create` · `#/mine` · `#/donate` · `#/admin` · `#/terms` · `#/privacy`

`#/mine` (**My page**) shows the fan-created profile you own — with its stats,
"created by you" badge and a one-tap ×1.5 self-boost. Profile pages also show
who created them (`created by @user`) and the creation date.

Legal modals pop from the footer; the **claim/verify modal** lives on every
seeded (fan-created) profile page.

---

## API surface (JSON, session cookies)

```
GET  /api/health                     GET  /api/me
POST /api/auth/signup|login|logout   POST /api/boost
GET  /api/profiles                   GET  /api/profiles/:slug
POST /api/profiles                   POST /api/profiles/:slug/claim
POST /api/daily-claim                POST /api/lucky-drop
GET  /api/tasks                      POST /api/tasks/:id/claim
POST /api/referral/share             GET  /api/referrals
GET  /api/leaderboard                GET  /api/feed
GET  /api/donations/methods          POST /api/donations
GET  /api/stream                     (SSE: hello/ping/presence/boost/claim/…)
POST /api/admin/login                GET  /api/admin/overview
POST /api/admin/announce|notify      POST /api/admin/season/settle
GET  /api/admin/claim-requests       POST /api/admin/claim-request
GET  /api/admin/users                POST /api/admin/reset
```

Admin console password: `grinbid-admin-dev` (override with `ADMIN_PASSWORD`).

---

## Legal & safety (baked in, not bolted on)

- **Coins are 100% free virtual coins** with zero cash value; no purchases,
  no Stripe, no pay-to-win. Prominent disclaimers on the SPA, legal pages and
  API responses.
- **Fan-created disclaimer + claim modal** on every seeded profile: pages are
  fan tributes, not endorsements; real owners can verify via moderator review.
- **Donations are voluntary and non-reward** (UPI / PayPal / Buy Me a Coffee /
  Razorpay) — never coins, boosts or perks.
- **Abuse guards**: 2-second boost cooldown, per-IP token-bucket rate limits
  (general / sensitive / boost), input sanitization everywhere, one community
  profile per user, anti-bot referral validation (same-IP flagging, per-day caps).

## Persistence & durability

`data/db.json` is a single JSON file. Every write goes to `db.json.tmp`,
is `fsync`'d, then atomically renamed over the live file (plus a directory
fsync). Writes are debounced (~150 ms) and serialized; `close()` forces a
flush. A corrupt file is backed up (`db.json.corrupt-<ts>`) and reseeded
instead of crashing.

## Audit

`npm run audit:loop` runs the required **20 consecutive flawless passes**:
zero-dependency & syntax checks → full test suite (economy math, atomic store,
rate limits, sanitization, SSE, legal pages) → live-server smoke of all 9 SPA
screens + legal views + `/api/stream`. Any failure resets the streak to 0.

## Deploy (free, mobile-friendly)

### Render — one click, best for a demo

[![Deploy to Render](https://render.com/images/deploy-to-render.svg)](https://render.com/deploy?repo=https://github.com/Kyabtao/grinbid)

Zero-dependency Node app + `render.yaml` blueprint: free plan, Node 22,
auto-generated `ADMIN_PASSWORD` / `SESSION_SECRET`, `/api/health` checks.
Full guide (one-click or manual, phone-friendly): **`deploy/RENDER.md`**.

> Caveat: Render's free disk is ephemeral — `data/db.json` reseeds on every
> redeploy. Fine for demos; the app recovers automatically.

### Oracle Cloud Always Free VM

See **`deploy/ORACLE-MOBILE.md`** for the full phone-only walkthrough:
Oracle Cloud Always Free VM + Termius. Once the VM exists, one pasted command
installs everything:

```bash
sudo bash -c "$(curl -fsSL https://raw.githubusercontent.com/Kyabtao/grinbid/arena/01a049f5-grinbid/deploy/oracle/setup.sh)"
```

- Installs Node 22 (ARM64-aware), clones Grinbid, creates a `systemd` service
  (auto-restart, start on boot), adds a keep-alive cron so Oracle doesn't
  reclaim an idle free instance, sets up log rotation.
- Then open **port 3000** in the VCN security list (steps in the doc).
- Always change `ADMIN_PASSWORD` (default `grinbid-admin-dev`) before going
  public.
