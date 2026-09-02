# Agent Control — sandbox

This repository is a **sandbox fork** of Agent Control with the V1 **approval inbox**.

It is **not production**.

- Production site: [agent-control.net](https://agent-control.net) (product code name: agent-guard). Do not ship sandbox experiments there.
- This app: dark-theme console + hold / Allow once / Always allow / Block.
- Trial here is **1 day (24 hours)**, matching production. Card checkout is not live in this sandbox.

## Run locally

1. Install Node 22+.
2. From the repo root: `npm install`
3. Start the app: `npm run dev`
4. Open the URL the script prints (preview binds `0.0.0.0:8080` in this workspace).
5. Sign in (email or configured providers) → you land on Overview. **Inbox** is in the left nav (and the mobile strip).

No `.env` is required for a local tour: the workspace falls back to an embedded database. Set `DATABASE_URL` only if you want a real Postgres.

Useful checks:

```bash
npm test
npm run typecheck
```

## How hold works

1. The agent calls `POST /api/v1/check` (or MCP `check_transfer`) with its API key **before** signing.
2. Policy can **block** (paused agent, denylist) or **hold** (unknown dest, over cap, velocity, first-time dest).
3. On hold: `must_abort` is true, `approval_id` + `poll_url` are returned. The agent must **not** sign. Poll `GET /api/v1/approvals/:id` or MCP `get_approval` until `allow` or `block`.
4. You decide in **Inbox**: Allow once, Always allow this address (adds to allowlist), or Block. Holds expire after **10 minutes** → agent must abort.

## Manual QA — Inbox

Sign in on a fresh account (demo agents + one sample hold are seeded).

- [ ] **Nav:** Inbox appears in the desktop sidebar and the mobile strip, with a count badge when holds are open.
- [ ] **Overview:** A “waiting for you” banner links to Inbox when holds exist.
- [ ] **Empty vs seeded:** After deciding the sample hold, Inbox shows the empty state. Scan chain / a pre-sign check can add more.
- [ ] **Allow once:** Request leaves Inbox. Agent poll returns `decision: allow`, `must_abort: false`. A **second** check to the same unknown address holds again (Allow once is not Always allow).
- [ ] **Always allow this address:** Request leaves Inbox; destination is on the agent allowlist; a later in-policy send to that address is `allow`.
- [ ] **Block:** Request leaves Inbox. Poll returns `block` / `must_abort: true`. Agent status can go critical.
- [ ] **Expiry:** Leave a hold untouched for 10 minutes. It disappears from Inbox; poll returns `block` (expired).
- [ ] **Paused / denylist:** Those still **block** immediately — they never appear in Inbox.
- [ ] **Trial copy:** Banner, billing, and landing say **24-hour / 1-day**, not 3-day.

## Out of scope for this sandbox

- Do not change production agent-guard.
- Do not add in-page wallet send (SSR / Phantom).
- Do not invent social proof or a fake Stripe checkout for Agent Control.
