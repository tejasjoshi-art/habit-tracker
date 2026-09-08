# Cause-Aware Habit Tracker (Workout + Reading)

A daily habit tracker for two habits — Workout and Reading — built around a
different assumption than most trackers: the problem usually isn't
motivation, it's **situational** (unpredictable time, energy, sleep). Instead
of just logging streaks, this app captures *why* a habit was missed and
surfaces patterns connecting misses to sleep and cause, so you can fix root
causes instead of guilting yourself into trying harder.

Built from `PRD-habit-tracker.md` as a V1 implementation. See that file for
the full spec — this README just covers running it.

## Run it locally

Requires [Node.js](https://nodejs.org/) 18+ (any recent LTS is fine).

```bash
npm install
npm run dev
```

Then open the URL it prints (usually `http://localhost:5173/`) in your
browser. Leave the terminal window open while you use the app — closing it
stops the local server.

To stop the server, press `Ctrl+C` in that terminal.

## Other commands

```bash
npm run build    # production build, output in dist/
npm run preview  # preview the production build locally
npm run lint     # run the linter
```

## Data & persistence

Everything is stored locally in your browser's `localStorage` — no backend,
no account, no sync (by design, see PRD section 2). Data persists across
restarts as long as you keep using the same browser on this machine. Clearing
site data / browser storage for `localhost` will reset it.

## Project structure

```
src/
  lib/
    model.js                 data model & constants
    storage.js                localStorage persistence
    dateUtils.js               date helpers
    suggestTimeWindows.js      schedule -> free/busy -> suggested habit windows
    computeInsights.js         the core differentiator: pattern/correlation logic
    motivationalMessages.js    data-driven + generic-fallback message pool
    rollover.js                end-of-day pending -> missed rollover
  components/
    EveningPlanning.jsx  HomeScreen.jsx  HabitCard.jsx
    MissReasonModal.jsx  WeeklyInsights.jsx  HistoryView.jsx
    NavBar.jsx  MotivationBanner.jsx
  App.jsx   main.jsx   index.css
```
