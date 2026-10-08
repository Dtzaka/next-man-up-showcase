<div align="center">

# 🏈 Next Man Up

### Real-time fantasy-football injury alerts that tell you the *exact* roster move to make.

A starter goes down 90 minutes before kickoff. You don't need another notification telling you he's hurt — you need to know **what to do about it**. Next Man Up watches your Sleeper roster, catches the injury, and surfaces the single best move: who to start, who to pick up, who to drop, what to stash on IR — each with a projected point delta.

<br/>

[![Live Demo](https://img.shields.io/badge/Live_Demo-next--man--up.vercel.app-000000?style=for-the-badge&logo=vercel&logoColor=white)](https://next-man-up.vercel.app)

<br/>

[![Next.js 16](https://img.shields.io/badge/Next.js_16-000000?style=flat-square&logo=nextdotjs&logoColor=white)](https://nextjs.org)
[![React 19](https://img.shields.io/badge/React_19-149ECA?style=flat-square&logo=react&logoColor=white)](https://react.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript_strict-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org)
[![Tailwind v4](https://img.shields.io/badge/Tailwind_v4-38BDF8?style=flat-square&logo=tailwindcss&logoColor=white)](https://tailwindcss.com)
[![Swift · SwiftUI](https://img.shields.io/badge/Swift_·_SwiftUI-F05138?style=flat-square&logo=swift&logoColor=white)](https://developer.apple.com/swift/)
[![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=flat-square&logo=supabase&logoColor=white)](https://supabase.com)
[![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white)](https://vercel.com)

<br/>

<img src="screenshots/03-dashboard.png" width="300" alt="Next Man Up dashboard" />

<sub><i>The dashboard — your roster, your risks, your next move, at a glance.</i></sub>

</div>

<br/>

---

## What it does

Most apps tell you *that* a player is hurt. By the time you've cross-referenced your bench, checked slot eligibility, and scanned the waiver wire, the window has closed. Next Man Up collapses that whole loop into one line: **here is the move, and here's how many points it's worth.** It detects, recommends, and explains — it never makes moves in Sleeper for you.

|  |  |
|---|---|
| 🔔 **Injury alerts** | A live feed of injury and status events that actually touch *your* roster — not the league-wide firehose. |
| 🔁 **Start / Sit** | Concrete swaps with projected **point deltas**, so you know exactly what the move is worth. |
| ➕ **Waiver pickups** | When your bench can't cover the gap, it surfaces the best available replacement. |
| ➖ **Drops & 🏥 Move-to-IR** | Recovers roster spots and frees an IR slot when a player qualifies. |
| ⚙️ **Channels + quiet hours** | Per-channel delivery preferences and quiet-hours gating, so 3 a.m. waiver news waits until morning. |

---

## Screenshots

<table>
  <tr>
    <td align="center"><img src="screenshots/03-dashboard.png" width="240" alt="Dashboard"/><br/><sub><b>Dashboard</b></sub></td>
    <td align="center"><img src="screenshots/04-moves.png" width="240" alt="Moves"/><br/><sub><b>Moves</b> · start/sit + point deltas</sub></td>
    <td align="center"><img src="screenshots/05-injuries.png" width="240" alt="Alerts"/><br/><sub><b>Alerts</b> · injury feed</sub></td>
  </tr>
  <tr>
    <td align="center"><img src="screenshots/06-settings.png" width="240" alt="Settings"/><br/><sub><b>Settings</b> · channels + quiet hours</sub></td>
    <td align="center"><img src="screenshots/02-onboarding.png" width="240" alt="Onboarding"/><br/><sub><b>Onboarding</b> · connect Sleeper</sub></td>
    <td align="center"><img src="screenshots/01-auth.png" width="240" alt="Auth"/><br/><sub><b>Auth</b></sub></td>
  </tr>
</table>

---

## Product status

Next Man Up is a shipped web and iOS product, built end to end by a solo developer.

| Capability | State | Notes |
|---|:---:|---|
| Web app — Next.js / React / TypeScript | ✅ **Live** | Available on [Vercel](https://next-man-up.vercel.app). |
| Native iOS app — Swift / SwiftUI | ✅ **Shipped on the App Store** | Native client sharing the backend with the web app. |
| Recommendation engine | ✅ **Implemented + tested** | Deterministic rankings and eligibility checks for start/sit, waiver, drop, and move-to-IR candidates, with LLM suggestions bounded to eligible candidates. |
| Shared TypeScript backend | ✅ **Live** | Supabase Postgres, Auth, and Row-Level Security support both clients. |
| Scheduled sync + push pipeline | ✅ **Implemented** | Scheduled jobs sync roster and player data, detect changes, generate recommendations, and queue notifications with delivery preferences. |
| Automated tests | ✅ **In place** | Coverage for recommendation logic and backend behavior. |

---

## Architecture

```
Sleeper API
     │  sync
     ▼
┌────────────────────────────────────────────────────────────┐
│  Supabase — Postgres + Auth + RLS                           │
│  players · rosters · snapshots · news_events ·              │
│  recommendations · alerts · device_tokens · prefs · …       │
└────────────────────────────────────────────────────────────┘
     │  pg_cron + pg_net drive the pipeline every 5–15m
     ▼
   sync  ──▶  detect  ──▶  recommend  ──▶  deliver
                                              │
                                              ▼
                                   Web Push · APNs · Email · SMS

        web (Next.js)  ◀──  RLS reads  ──▶  iOS (SwiftUI)
```

- **Cron drives everything.** pg_cron schedules the pipeline; pg_net calls guarded Next.js endpoints. The service role is used *only* by cron — never by the clients.
- **Clients read through RLS.** The web app and the iOS app each read their own data through Row-Level Security. No client ever holds elevated privileges.
- **The engine is deterministic and tested.** Scoring, slot eligibility, and candidate ranking are covered by automated tests. LLM suggestions are bounded to eligible candidates.

---

## Tech stack

| Layer | Tools |
|---|---|
| **Web** | Next.js 16 (App Router) · React 19 · TypeScript (strict) · Tailwind v4 · shadcn/ui |
| **iOS** | Swift · SwiftUI · Keychain · APNs |
| **Backend** | Supabase — Postgres · Auth · RLS · pg_cron · pg_net |
| **Data & delivery** | Sleeper public API · Web Push (VAPID) · Resend · Twilio |
| **Infra** | Vercel |

---

## Source

The full source — the Next.js web app, the native SwiftUI client, and the Supabase backend (schema, RLS, recommendation engine, and the scheduled pipeline) — lives in a **private repository**. Happy to walk through the architecture or code on request.

---

<div align="center">

Built by **Denis Tzaka**.

<sub>Sleeper is a trademark of Blitz Studios, Inc. Next Man Up is an independent project and is not affiliated with or endorsed by Sleeper.</sub>

</div>
