# Handoff — Sleep Debt & Energy Curve (public release)

## What this is

A single-file web app that turns Apple Health sleep data into two things: a rolling
sleep debt figure and a predicted energy curve for the day. No account, no server,
no analytics. Data is pasted in by the user and stored in their own browser.

It exists because the paid apps in this space (Rise, Sleep Cycle) compute two fairly
simple things behind a subscription, and anyone with an Apple Watch already owns the
underlying data.

**Source:** `public/index.html` — vanilla JS, no dependencies, no build step. Ships as-is.
`public/README.md` and `public/LICENSE` go in the repo root alongside it.

---

## Repo shape

```
/
  index.html          ← from public/index.html
  README.md           ← from public/README.md
  LICENSE             ← from public/LICENSE (MIT)
  og.png              ← social preview, 1200×630 (still to make)
```

Vercel deploys this as a static site with zero config. No `vercel.json` needed
unless you want a custom 404.

---

## Already done since the first draft

- **Empty state** built: what it is, a privacy line, the five Shortcut steps, and the
  Large Amounts of Data warning called out separately.
- **Sample data** button. `makeSample()` generates 14 nights relative to today, so it
  never goes stale. Realistic spread: 4.9–7.8h durations, ~13% deep / 20% REM / 66% core,
  short awakenings, and one night with a 52-minute awake block so the external-wake
  toggle is discoverable. Loading it sets `state.sample`, which shows a dismissible
  banner; pasting real data clears the flag.
- **Clear all data** button, and a separate Clear for the sample.
- **Responsive chart.** The viewBox now switches to 430×300 under 620px wide. At the old
  760-wide viewBox the chart scaled to ~50% on a phone and the labels were ~6px.
- Meta description and Open Graph tags.
- **Bug fix, applies to both versions:** `ideal` bedtime double-counted the circadian
  offset (`circ(wake)+24`), so `Math.max(ideal, onset-0.5)` always picked `ideal` and the
  gradual 30-minutes-earlier step never fired. `toB` in `habits()` was landing on a
  workable number only by accident and now normalises negatives. Verified against both
  real and sample data.
- README and MIT LICENSE written.

Browser-tested at 390px and 1100px: no console errors, no horizontal scroll, sample loads,
habit ticks persist, and the external-wake toggle moves that night from 87% to 99%.

## Still to do before linking it publicly

### 1. The Shortcut (the real bottleneck)

The README has the five actions written out, but building it by hand takes ~15 minutes
and the variable picker is genuinely confusing. Publish an iCloud Shortcuts link and put
it at the top of the setup section.

Keep the written steps as a fallback — iCloud links break when Apple rotates them.

### 2. Fill in the live URL

`README.md` has `**[Open it →](#)**` near the top. Point it at the deployed URL.

### 3. Optional

- `og.png` at 1200×630 for the social preview
- A custom 404

## Do not change

**The algorithm.** It was validated against real data and a real subjective report.
Specifically:

- `ANCHORS` — the 14 control points defining the energy curve, in hours after wake.
  Smoothstep interpolated. Peak around wake+3.2, dip at wake+8.8, second wind at
  wake+12.5. These match the standard two-process model landmarks.
- `estimateNeed()` — mean of the longest 20% of nights, clamped 6.5–9.5h. Do not
  substitute a fixed 8h.
- Debt — sum of `max(0, need − asleep)` across the last 14 nights. Not averaged.
- `dFactor()` — how debt flattens the curve. Floors at 0.42 so the chart never
  collapses to nothing.
- Habit times — all offsets from the user's own wake time and target bedtime.
  Nothing is a fixed clock time. This is the point of the feature.
- Target bedtime — 30 minutes earlier than their median, floored at what a full
  night actually requires. Deliberately gradual; an aggressive target fails on night
  one and the user stops trusting the app.

**Local-only storage.** No telemetry, no error reporting, no CDN fonts, no
third-party anything. This is the product's main claim over the paid apps. Do not
add a dependency that phones home.

**Night stitching.** Segments group into a night when the gap is ≤ 3 hours.
Periods under 2.5h asleep are discarded as naps. Awake and In Bed segments are
excluded from time asleep.

---

## Code map

| Function | Does |
| --- | --- |
| `parseSegments` | text → array of `{start, end, stage, hours}` |
| `buildNights` | segments → nights, 3h gap rule, captures `longAwake` (blocks ≥15min) |
| `estimateNeed` | longest-20% mean, clamped |
| `nightEff` | efficiency, minus long awake blocks on externally-flagged nights |
| `analyse` | the whole summary object: debt, score, wake, onset, target, stages |
| `energyAt` | smoothstep over `ANCHORS` |
| `habits` | seven windows derived from wake and target bedtime |
| `makeSample` | 14 nights of synthetic segments, generated relative to today |
| `chart` | the SVG: bands, curve, rested ghost line, habit lane, now marker |
| `render` | full re-render; state changes call it again, no diffing |

State shape:

```js
{ text: "<raw pasted segments>",
  need: null,                 // null = auto-estimate
  done: { "2026-09-15": ["light","caff"] },
  ext:  { "2026-09-15": 1 },  // nights woken externally
  sample: false }             // true while showing generated data
```

`HOST` detects `window.storage` (Claude artifact runtime) and falls back to
`localStorage`. Keep the fallback — it's what makes the same file work in both
places.

---

## Acceptance checks

- Empty state renders with no console errors and offers sample data
- Sample data loads in one tap and produces a full day view
- Pasting ~330 segments produces 14 nights, no duplicates, no missing nights
- Time asleep is always less than time in bed
- Tapping a night toggles external and raises only that night's efficiency
- Habit ticks persist across reload and roll over at local midnight, not UTC
- Works at 375px wide; no horizontal scroll
- Dark and light system themes both legible

---

## Out of scope for v1

Accounts, sync, notifications, Android or Google Fit, charts beyond the single day
view, and any form of data export. Ship the small thing.
