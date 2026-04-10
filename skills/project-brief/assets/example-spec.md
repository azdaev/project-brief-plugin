# kettlebell coach — technical specification

## overview
kettlebell coach is a personal training companion for hobbyist lifters who train alone in a garage gym without a coach. it generates a daily workout, records the session as you go, and surfaces weekly volume trends so the lifter can adjust intensity without spreadsheets.

primary user: a 32-year-old software engineer with a single 24kg kettlebell, training 4x/week in 30-minute slots, wanting structure without a human coach.

## user stories

- as a lifter, i can open the app and see today's prescribed session in under 5 seconds, so i don't lose momentum
- as a lifter, i can record reps/weight/RPE per set as i go, so i don't have to remember the whole session afterward
- as a lifter, i can see last week's total volume per movement pattern, so i know whether to push or deload
- as a lifter, i can override any prescribed set, so the app adapts to how i actually feel that day
- as a lifter, i can mark a session as skipped with a reason, so the generator accounts for it next time

## features (v0.1)

### 1. daily session view
- landing screen shows today's session: movements, sets x reps, target weight, RPE target
- each set has a tap target to mark complete; long-press to adjust weight/reps
- timer between sets, configurable per movement
- acceptance: opening the app on a training day takes ≤2 taps to start logging

### 2. session logging
- per-set fields: actual reps, actual weight, actual RPE
- "felt heavy"/"felt light" quick flags
- session notes textarea
- auto-saves to local storage, syncs on reconnect
- acceptance: a 30-min session can be fully logged with the phone on the floor, one-thumb

### 3. session generator
- rules engine (not ML) — inputs: last 4 weeks of logs, available equipment, target split
- generates a 7-day microcycle, refreshes every monday
- user can regenerate a single day without disturbing the rest of the cycle
- acceptance: generated sessions respect: (a) no two heavy days in a row, (b) ≥48h between sessions targeting the same movement pattern

### 4. weekly trends
- single screen: volume (sets × reps × weight) per movement pattern, last 4 weeks
- sparkline per pattern
- no charts, no configurability — one opinionated view
- acceptance: renders in ≤200ms from cached data

## data model

```
user
  id, email, created_at, timezone, equipment (json)

movement
  id, name, pattern (squat|hinge|press|pull|carry|core), unit (kg|lb|bodyweight)

session
  id, user_id, date, status (scheduled|in_progress|done|skipped), generated_by, notes

set
  id, session_id, movement_id, order_index, prescribed_reps, prescribed_weight, prescribed_rpe,
  actual_reps, actual_weight, actual_rpe, felt (heavy|light|normal), completed_at

cycle
  id, user_id, week_start, plan (json, the full 7-day prescription)
```

indices: session(user_id, date), set(session_id), cycle(user_id, week_start).

## api / routes

```
GET  /api/session/today           → today's session with sets
POST /api/session/:id/set/:n      → update one set's actuals
POST /api/session/:id/complete    → mark done, lock
POST /api/session/:id/skip        → mark skipped with reason
POST /api/session/:id/regenerate  → regenerate just this day
GET  /api/trends/volume?weeks=4   → weekly volume per pattern
```

all routes require authenticated user; magic link for auth.

## technical stack

- **language:** go 1.22
- **web framework:** standard library `net/http` + chi router
- **database:** sqlite with WAL mode (single-user product, no need for postgres)
- **migrations:** goose
- **frontend:** htmx + templ + vanilla js for the set logger
- **hosting:** fly.io, single region, 256mb instance
- **auth:** email magic link via postmark
- **background jobs:** none — cycle generation runs on first request after monday 00:00 user-local

## ui / ux

three screens total:
1. **today** — the session view, full-screen, one thumb
2. **log** — per-set logger, stays on-screen during the workout
3. **trends** — weekly volume, scrollable, no interaction

design feel: calm, focused, not "gym app." generous spacing, serif headings, no progress bars flashing green. the lifter is mid-workout — nothing should distract.

## non-functional requirements

- **performance:** today view ≤200ms from cold cache; set logging feedback ≤50ms
- **offline:** full session can be logged offline; syncs on reconnect
- **privacy:** no third-party analytics, no tracking, logs stay in user's sqlite row
- **accessibility:** all interactive elements ≥44pt hit area; respects reduced motion
- **scale:** designed for 1–500 users; will be rewritten before it needs more

## out of scope (v0.1)

- social features (sharing, friends, leaderboards) — distracts from the core loop
- video form checks — camera + upload + storage is a separate project
- wearable integration — heart rate, HRV, apple watch. maybe v0.2
- nutrition tracking — scope creep, use a different app
- multi-user / coach accounts — single-lifter product for now
- mobile native app — web works fine on phones, saves build complexity

## open questions

- **cycle generator heuristics** — start with hand-tuned rules, but what's the feedback signal for "this plan is too hard/easy"?
- **volume calculation** — do bodyweight movements count toward total volume, and if so, at what weight?
- **timezone edge cases** — if a user travels mid-cycle, does the plan shift or stay on the origin timezone?
