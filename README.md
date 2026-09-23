# Sleep debt & energy curve

Turn Apple Health sleep data into two things: how much sleep you owe, and when your
energy will actually peak and dip today.

One HTML file. No framework, no build step, no server, no account. Every calculation
runs in your browser and your data never leaves the device.

**[Open it →](https://shafakat.studio/sleepdebt)** &nbsp;·&nbsp; there's a **See it with sample data** button, so you can
look before setting anything up.

## Why

Paid sleep apps compute two fairly simple things behind a subscription:

- **Sleep debt** — the gap between the sleep you need and the sleep you got, summed
  over the last 14 nights.
- **A circadian energy curve** — predicted from the midpoint of your sleep, producing
  the morning grogginess window, the late-morning peak, the afternoon dip, and the
  evening second wind.

If you own an Apple Watch, the underlying data is already yours. This works both out
and draws your day around them.

## What it shows

A dashboard of cards, in the order you'd actually want them:

- **Your score** — out of 100, tied to your debt, with the hours owed, your average night,
  and the sleep need everything is measured against
- **Tonight** — the time to be asleep by and how long that is from now, on a timeline
  running melatonin → screens down → asleep → wake
- **Energy today** — your predicted curve against a dotted line showing the same day fully
  rested, peak and dip bands, a marker for right now, and a lane showing where the habit
  windows fall
- **Today's habits** — seven windows (morning light, last caffeine, nap, training, last
  alcohol, last meal, screens down), each tickable, each showing whether it has passed, is
  open now, or is the next one up. All derived from your own wake time and target bedtime,
  so they move as your schedule moves.
- **Last 14 nights** — each night as a bar against your need line, so short nights are
  obvious at a glance
- **What those hours were made of** — a deep/REM/core breakdown and how much of your time
  in bed was actually asleep
- **Recent nights** — a per-night list where you can flag a night that something outside
  woke you. The lost sleep still counts toward debt; it just stops dragging your efficiency
  figure down.

It's one column on a phone and a grid on a wider screen.

## Getting your own data in

You need an iOS Shortcut that reads Apple Health on-device and copies the last 14 nights
to your clipboard. Five actions:

1. **Find Health Samples where** — All of: Type is Sleep; Start Date is in the last 14 days
2. **Repeat with each item in** Health Samples
3. **Text** — `Repeat Item · Start Date` `|` `Repeat Item · End Date` `|` `Repeat Item · Value`,
   with both dates set to custom format `yyyy-MM-dd HH:mm`
4. **Combine Text** — Repeat Results, with New Lines
5. **Copy to Clipboard**

Then open the app and tap **Paste from clipboard**. On a fresh install that button is on
the opening screen; once you have data in, it moves behind the settings button in the top
right, along with setting your sleep need by hand, re-estimating it, and clearing
everything.

### The one thing that will trip you up

Turn on **Settings › Shortcuts › Advanced › Allow Sharing Large Amounts of Data** before
running it. Without this, iOS refuses outright with a message about trying to share N
Health items.

### Expected format

One segment per line. Awake and In Bed segments are read and excluded from time asleep.

```
2026-09-13 00:21 | 2026-09-13 00:34 | Core
2026-09-13 00:34 | 2026-09-13 01:04 | Deep
2026-09-13 01:04 | 2026-09-13 01:11 | Core
```

Any source that produces this shape works. Apple Watch gives you Core/Deep/REM/Awake;
an iPhone-only night comes through as a single `Asleep` block, which still counts toward
totals but has no stage breakdown.

## How the numbers work

- **Sleep need** is estimated from the mean of your longest 20% of nights, clamped to
  6.5–9.5h. It is not a fixed 8 hours, and it firms up as your history grows.
- **Debt** is the sum of `max(0, need − asleep)` across the last 14 nights. Not averaged.
  Under about 5 hours is where it stops noticeably affecting you.
- **Nights** are built by grouping segments with gaps of 3 hours or less. Periods under
  2.5 hours asleep are treated as naps and excluded.
- **The curve** is a smoothstep interpolation over 14 control points expressed in hours
  after wake, matching the standard two-process model landmarks. Debt flattens the whole
  curve rather than shifting it.
- **Tonight's target** is 30 minutes earlier than your median bedtime, floored at whatever
  a full night actually requires. It tightens as your median moves earlier. A target you
  miss on the first night just teaches you to ignore the app.

## Privacy

There is no backend. No analytics, no error reporting, no fonts or scripts loaded from
anywhere. Your sleep data is held in `localStorage` on the device you pasted it into and
is never transmitted. **Clear all data**, in the settings sheet, wipes it.

The trade-off: phone and laptop keep separate copies. Since the Shortcut always exports
the last 14 days, one paste rebuilds everything on either one.

## Running it

It's a single static file. Open `index.html` locally, or deploy the directory to any
static host. No build, no dependencies, no configuration.

## Limitations

- Apple Health only. Any source that can produce the line format above will work, but
  nothing else is wired up.
- The energy curve is a population model fitted to your timings, not a model learned from
  your own reported energy.
- Habit windows are derived from sleep timing alone. It knows nothing about your calendar.

## Licence

MIT.
