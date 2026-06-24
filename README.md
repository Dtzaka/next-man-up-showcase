# 🏈 Next Man Up

**Real-time fantasy football injury alerts — with the exact move you should make next.**

Next Man Up watches your [Sleeper](https://sleeper.com) lineups and tells you who to start, add, drop, or move to IR the moment an injury, inactive, or breaking-news event hits your roster — before your league-mates react. It detects, recommends, and explains. It never touches your team for you.

🔗 **Live app:** [next-man-up.vercel.app](https://next-man-up.vercel.app) · 📱 Native iOS app · 🖥️ Web dashboard

> This is a **product showcase**. The source code is private — happy to walk through it on request.

---

## What it does

You connect your Sleeper username once. From then on, a backend pipeline runs around the clock:

1. **Detects** roster-relevant changes — injury-status flips, inactives, depth-chart and news updates — by diffing snapshots of every player you roster.
2. **Recommends** the actual move: start a benched player over an injured starter, pick up a backup off waivers, drop dead weight, or stash someone on IR — each scored, severity-ranked, and explained in plain English.
3. **Delivers** it to you — web push today, with APNs (iOS), email, and SMS wired in — gated by your per-league alert preferences and quiet hours.

---

## The iOS app

<table>
  <tr>
    <td align="center"><img src="screenshots/03-dashboard.png" width="240"><br><sub><b>Dashboard</b> — leagues, urgency, top moves</sub></td>
    <td align="center"><img src="screenshots/04-moves.png" width="240"><br><sub><b>Moves</b> — start/sit with point deltas</sub></td>
    <td align="center"><img src="screenshots/05-injuries.png" width="240"><br><sub><b>Alerts</b> — live injury feed</sub></td>
  </tr>
  <tr>
    <td align="center"><img src="screenshots/06-settings.png" width="240"><br><sub><b>Settings</b> — channels & quiet hours</sub></td>
    <td align="center"><img src="screenshots/02-onboarding.png" width="240"><br><sub><b>Onboarding</b> — connect Sleeper</sub></td>
    <td align="center"><img src="screenshots/01-auth.png" width="240"><br><sub><b>Auth</b></sub></td>
  </tr>
</table>

A native **SwiftUI** app that talks to the same backend as the web app under the same row-level-security rules — one source of truth, two clients.

---

## Features

- **Real-time detection** — snapshot-diff pipeline over the full ~12k-player Sleeper map, idempotent and deduplicated.
- **A real recommendation engine** — value scoring (scarcity, depth chart, trending, matchup), slot-eligibility, start/sit, waiver pickup, drop, and move-to-IR — each with a severity, a confidence, an expected point delta, and a human explanation.
- **Multi-channel alerts** — Web Push (live), plus APNs, email, and SMS — fanned out per recommendation and gated by per-league alert levels.
- **Two clients, one backend** — a Next.js web dashboard and a native SwiftUI iOS app, both reading Supabase directly under RLS.
- **Scheduled, serverless, free-tier** — the whole sync→detect→recommend→deliver loop runs on Postgres `pg_cron` hitting guarded endpoints every few minutes.

---

## How it works

```
Sleeper API
    │
    ▼
┌─────────────────────── Supabase (Postgres + Auth + Row-Level Security) ───────────────────────┐
│   players · rosters · snapshots · news_events · recommendations · alerts · device tokens       │
└───────────────────────────────────────────────────────────────────────────────────────────────┘
    ▲                         ▲                                   ▲
    │ pg_cron (every 5–15m)   │ RLS reads / user writes           │ service-role (cron only)
    │                         │                                   │
 sync players/leagues/     Web app (Next.js)              detect → recommend → send-alerts
 trending/availability     iOS app (SwiftUI)              (the recommendation engine)
```

- **Detect → recommend → deliver** is a chain of guarded serverless jobs, scheduled from Postgres so it runs on a free hosting tier at real-time cadence.
- **Security model:** two trust levels — a cookie/JWT-bound client scoped to the user by RLS, and a service-role key used *only* by the cron jobs to write shared system tables.

---

## Tech stack

| Layer | Tech |
|---|---|
| **Web** | Next.js 16 (App Router), React 19, TypeScript (strict), Tailwind v4, shadcn/ui |
| **iOS** | Swift / SwiftUI, native, Keychain-backed sessions, APNs |
| **Backend** | Supabase — Postgres, Auth, Row-Level Security, `pg_cron` + `pg_net` |
| **Data** | Sleeper public API · Web Push (VAPID) · Resend · Twilio |
| **Infra** | Vercel (app + functions) · scheduled from Supabase |
| **Quality** | Vitest unit suite · strict typecheck · adversarial pre-merge code review |

---

<sub>Built by Daniel Tzaka. Source private — available for review on request.</sub>
