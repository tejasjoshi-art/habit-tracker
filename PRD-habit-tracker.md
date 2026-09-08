# PRD: Cause-Aware Habit Tracker (Workout + Reading)

## 1. Overview

A daily habit tracking app for two habits — **workout** and **reading** — built around one core insight: most habit trackers assume the user's problem is motivation (streaks, gamification). This app is designed for users whose real barrier is **situational**: unpredictable daily time, energy, and sleep. Instead of just logging completion, the app captures *why* a habit was missed and surfaces patterns (e.g. "you skip workouts on low-sleep days") so the user can fix root causes, not just guilt themselves into trying harder.

This document is a build spec intended for an AI coding agent (Claude Code) to implement a working V1 web app.

## 2. Goals (V1)

1. Let a user track two fixed habits daily: Workout, Reading.
2. Capture daily sleep (self-reported) and a flexible time-window per habit, instead of fixed-time reminders.
3. On a missed habit, prompt a one-tap reason: No time / Low energy / Forgot.
4. Show a weekly insights view connecting misses to sleep and cause data.
5. Make every daily interaction (log complete, log miss reason, log sleep) completable in under ~5 seconds.

**Explicitly out of scope for V1:** wearable integrations, gamification/streak badges, social features, unlimited custom habits, AI-generated scheduling (use simple rule-based logic instead), user auth/multi-device sync (single local user is fine for V1).

## 3. Target User

Students / young professionals who want to work out and read daily but fail due to time, energy, and sleep constraints rather than lack of motivation. A single miss doesn't demotivate them; an invisible pattern of repeated misses does.

## 4. Core User Flows

### 4.1 Evening Planning (done the night before, for the next day)
1. User opens app in the evening (or whenever convenient) and taps "Plan Tomorrow."
2. Sleep slider: "How many hours did you sleep last night?" (used to contextualize today's performance, one tap/drag).
3. Schedule input: user lists tomorrow's busy blocks in plain terms — a short list of entries like "9–11 Class," "2–5 Work," each just a label + rough time range. Keep this fast: no calendar-grid UI, just an add-a-row list capped at ~5 entries.
4. App computes free windows from the busy blocks (Morning / Afternoon / Evening buckets) and **suggests** a time-window for each habit (Workout, Reading) based on which buckets are free. User can accept the suggestion with one tap or override it manually.
5. App shows one short motivational message tied to the plan (see 4.1a) and lands on Home.

**Fallback:** if the user skips planning the night before, Home screen shows a lightweight version of this flow first thing in the morning instead — same steps, just later. Planning is never blocked; a "Skip for today" option always falls back to a default Morning/Afternoon/Evening pick with no schedule input.

### 4.1a Motivational Messages
- Shown at two moments: (1) right after Evening Planning is completed, and (2) right after a habit is marked Done.
- Default style: **effort-under-difficulty framing**, generated from the user's own recent data rather than generic quotes — e.g. "You're planning around a packed day tomorrow — that's exactly the kind of day this used to get skipped." or "3rd workout this week despite two low-sleep days — that's real consistency." This ties motivation to the user's actual situational pattern rather than generic hype, matching the fact that a single miss doesn't demotivate this user, but seeing effort recognized does.
- Configurable message pool should also include a small set of general aspirational lines (e.g. short quotes about discipline/consistency) as a fallback for days with no notable pattern yet (e.g. day 1–2, no data). Keep this pool short (~10–15 lines) and rotate them so they don't repeat back-to-back.
- Never show a motivational message alongside a miss — no "better luck tomorrow" copy on a miss; the miss-reason capture (4.3) stays neutral/observational, not motivational.

### 4.2 Home Screen (main daily view)
- Shows today's two habits as cards: Workout, Reading.
- Each card shows: habit name, planned time-window (from Evening Planning), a "Mark Done" button.
- If sleep logged is low (below a configurable threshold, default 6 hrs) AND the habit is Workout, show an inline nudge: "Low sleep today — consider a lighter session or moving this to evening."
- If a habit's planned time-window has passed and it's not marked done, the card visually flags it as "at risk" and offers a "Mark Done" or "Didn't happen" action (leads to 4.3).

### 4.3 Miss Reason Capture
- Triggered when user taps "Didn't happen" on a habit, or at end-of-day rollover for any habit still unmarked.
- Single screen: "Why did [Workout/Reading] not happen today?"
- Three tap options: **No time**, **Low energy**, **Forgot**.
- One tap saves and returns to Home. No free text required (optional free-text "add detail" field, collapsed by default, never required).

### 4.4 Weekly Insights
- Accessible via a tab/nav item.
- Shows, per habit, over the last 7 days:
  - Completion count (e.g. "Workout: 4/7 days")
  - Miss reasons breakdown (e.g. "No time: 2, Low energy: 1")
  - Correlation callout if pattern exists: e.g. "3 of your 3 missed workouts happened on days you slept under 6 hrs." (simple rule-based check: if ≥2 misses share a common factor — low sleep day, or same reason tag — surface it as a callout.)
- If insufficient data (<3 days logged), show an empty/encouraging state instead of forcing an insight.

### 4.5 Habit History (simple calendar/list view)
- Optional simple view: last 14–30 days, per day showing status per habit (Done / Missed + reason / Not yet logged).
- Keep it simple — a table or grid is fine, no need for a fancy calendar widget in V1.

## 5. Data Model (suggested)

```
DailyLog {
  date: string (YYYY-MM-DD)
  sleepHours: number | null
  scheduleBlocks: ScheduleBlock[]   // entered during Evening Planning, capped ~5
  habits: {
    workout: HabitEntry
    reading: HabitEntry
  }
}

ScheduleBlock {
  label: string          // e.g. "Class", "Work"
  startHour: number       // 0-23, rough hour is enough
  endHour: number
}

HabitEntry {
  timeWindow: "morning" | "afternoon" | "evening" | null
  windowSource: "suggested" | "manual" | "default"   // was the window auto-suggested from schedule, user-picked, or fallback default
  status: "pending" | "done" | "missed"
  missReason: "no_time" | "low_energy" | "forgot" | null
  missDetail: string | null   // optional free text
}
```

Storage: for V1, local persistent storage is sufficient (no backend/auth required). If built as a Claude artifact, use the `window.storage` key-value API (personal/non-shared) with a key pattern like `dailylog:YYYY-MM-DD`. If built as a standalone app, local storage / a simple local JSON/SQLite file is fine.

## 6. Screens Summary

| Screen | Purpose |
|---|---|
| Evening Planning | Log sleep + tomorrow's schedule blocks; get suggested time-windows per habit + a motivational message |
| Home | View today's habits, mark done/missed, see nudges |
| Miss Reason | Quick 1-tap reason capture on a miss |
| Weekly Insights | Pattern callouts connecting misses to sleep/cause |
| History | Simple log of past days per habit |

## 7. Design Principles for the Agent Building This

- **Speed over completeness**: every daily interaction must be doable in 1–2 taps. Do not add multi-field forms.
- **No guilt-based UI**: no red "broken streak" language, no shaming copy. Tone should be neutral/observational (e.g. "3 misses this week, mostly low-sleep days" not "You failed 3 times").
- **No streak counters or gamification** — this app deliberately avoids that pattern; do not add points, badges, or XP.
- **Motivation is earned/data-driven, not generic hype** — prefer messages that reference the user's own recent effort/pattern over stock motivational quotes; keep a small generic-quote fallback pool only for early days with no data yet. Never pair a motivational message with a miss.
- **Insights only when there's enough data** — don't force a pattern out of 1–2 data points.
- **Mobile-first, minimal, clean UI** — this is a personal daily-use tool, not a dashboard-heavy product. Favor a single home screen with a lightweight nav to Insights/History.

## 8. Acceptance Criteria (V1 "done")

- [ ] User can complete Evening Planning (sleep + up to 5 schedule blocks) in under 30 seconds.
- [ ] App suggests a time-window per habit based on free blocks derived from the entered schedule, and the user can override it with one tap.
- [ ] If planning is skipped, Home screen offers the same flow the next morning, or a "Skip for today" default.
- [ ] A motivational message appears after planning and after marking a habit Done, using recent-data framing when data exists, generic fallback otherwise — never on a miss.
- [ ] User can mark a habit Done or Missed from the Home screen.
- [ ] Marking a habit Missed always prompts the 3-option reason tag.
- [ ] Low-sleep nudge appears on the Workout card when sleep < 6 hrs (configurable constant).
- [ ] Weekly Insights correctly aggregates completion counts and miss reasons for the last 7 days.
- [ ] Weekly Insights surfaces a sleep-correlation callout only when the rule condition is met (≥2 misses sharing the same contributing factor).
- [ ] All data persists across sessions (survives a page refresh / app reopen).
- [ ] No login/auth required for V1 — single local user.

## 9. Suggested Build Approach for Claude Code

- Single-page app (React recommended) with client-side state + local persistence.
- Component breakdown: `EveningPlanning`, `HomeScreen`, `HabitCard`, `MissReasonModal`, `WeeklyInsights`, `HistoryView`.
- Keep the correlation logic in one small utility function (e.g. `computeInsights(logs: DailyLog[])`) so it's easy to test and extend later (this is the core differentiator feature — keep it clean and isolated).
- Keep the schedule-to-window suggestion logic in its own small utility (e.g. `suggestTimeWindows(scheduleBlocks)`) — simple rule-based free/busy bucketing (Morning/Afternoon/Evening), not a scheduling algorithm.
- Keep the motivational-message pool and selection logic in its own small module (e.g. `getMotivationalMessage(logs: DailyLog[])`) so the data-driven vs. generic-fallback logic is easy to tweak independently of the UI.
- Start with Workout + Reading hardcoded as the only two habits; structure the data model so adding a third habit later is a small change, not a rewrite.
