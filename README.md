# CourseApp

A single-file web app for students that tracks grades across courses and warns you early when your engagement drops.

- **Grade tracker** – add courses with a syllabus-style grading scheme and see your worst-case / best-case total at any moment, plus the exam score you need to pass or keep a scholarship.
- **Engagement self-check** – log how a class went (attendance, participation, understanding, or whether you watched the recording). The app compares each check-in with your personal baseline and flags a noticeable drop.

No framework, no build step, no backend. Everything lives in one `index.html`.

## Features

| Area | What it does |
| --- | --- |
| Courses | Semester dates, class weekdays, auto-generated class schedule, per-course pass / scholarship thresholds |
| Grading | Components such as midterm, quiz, homework, lab, project. Add one specific item or a repeating series (e.g. 10 homeworks, every Tuesday from week 2) |
| Grade locking | A grade can be entered only on the day the work takes place (set by week + weekday) |
| Result range | Worst / best total while grades are incomplete, and the exam score needed for each threshold |
| Check-ins | Only past classes that are not yet checked in can be logged, so no duplicates and no future days |
| Engagement chart | Interactive: tap a point to see week, date, score and difference from baseline; step with arrows; 7 / 14 / all range |
| Demo date | Set "today" in Settings to show how the app behaves over a semester |
| Theme | Auto / light / dark, translucent glass UI on a gradient background |

## Run it

Open `index.html` in any modern browser. Internet is only needed for Google Fonts; without it the app falls back to system fonts.

### Deploy on GitHub Pages

1. Put `index.html` (and this README) in a repository.
2. **Settings → Pages**, choose the `main` branch, root folder.
3. The site appears at `https://<username>.github.io/<repo>/`.

## How it works

```
index.html
├── CSS            design tokens (light/dark), layout, components
├── Store          read/write JSON in localStorage ("courseapp_v3")
├── CalcEngine     grade ranges and "score needed on the exam"
├── BaselineEngine check-in score, personal baseline, drop detection
└── App            state, navigation, actions, render_<view>() functions
```

**Rendering.** All state is in the `App` object (`view`, `activeCourseId`, `courseTab`, …). Every action updates the state and calls `App.render()`, which asks the current `render_<view>()` for an HTML string and puts it into `<main>`. Buttons call `App` methods through inline `onclick` handlers.

**Screens.** Home, Courses, Check-in, Settings, plus a course page with tabs: Overview, Grades, Engagement, Classes, Formula. A bottom navigation bar switches between the main screens.

### Data model

```js
course = {
  id, name, startDate, endDate, weekdays, totalWeeks,
  sessions: [{ date, week, dow }],
  split: { prefinal, final },                 // e.g. 60 / 40
  thresholds: { close, scholarship },
  groups: [{ type, label, count, totalWeight,
             entries: [{ score, week, dow }] }],
  session: { decided, date, score },          // the exam
  checkins: [{ date, attended, participation, understanding, watched }]
}
```

### Grade calculation

- Each item weighs `group weight / number of items`.
- Graded items add their known points; ungraded ones add between 0 and their weight.
- **Worst case:** everything ungraded and the exam are 0. **Best case:** everything ungraded is maxed.
- `Total = prefinal% × current points + final% × exam`.
- The needed exam score is found by solving that equation for each threshold.
- Status colour: green if the worst case is at or above the scholarship threshold, red if the best case is below the pass threshold, amber otherwise.

### Engagement score

| Situation | Score |
| --- | --- |
| Attended | `40 + participation × 30 + understanding × 30` (1–5 scales mapped to 0–1) |
| Absent, watched the recording | `15 + clarity × 25` (max 40) |
| Absent, did not watch | `0` |

- **Baseline** = average of the previous 4 check-ins (the first baseline appears after 5).
- **Drop** = today's score is at least 35 % below the baseline.
- A skipped day is simply not logged and never counts against you.

### Dates

`App.today()` returns the demo date when one is set, otherwise the real date. All dates are handled in local time (`isoDate()`), and `migrate()` upgrades data saved by older versions.

## Privacy

Everything you enter is stored only in your browser's `localStorage` under the key `courseapp_v3`. The app has no server, no accounts, no analytics and sends no data anywhere. The only external request is the Google Fonts stylesheet.

Clearing browser data, or opening the app on another device or domain, starts with an empty app.

## Limitations

- No sync between devices and no backup or export yet.
- The whole screen re-renders on each action, so inputs outside the add-course name field lose focus on re-render.
- No automated tests.

## Ideas

- JSON export / import for backup
- Per-course notifications when engagement drops
- Accounts and cloud sync
- Native iOS client using the same calculation logic

## Tech

HTML, CSS and vanilla JavaScript. Fonts: Fraunces, Plus Jakarta Sans, IBM Plex Mono (Google Fonts).
