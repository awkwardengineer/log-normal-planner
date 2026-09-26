# Log-Normal Monte Carlo Planner

A single-file project planner that uses log-normal Monte Carlo simulation to estimate timelines (and related uncertainty) from uncertain task estimates.

## Goals

- Fully encapsulated: one HTML file containing markup, styles, and all application JavaScript.
- No build step or toolchain. Open the file in a browser and it works.
- Tailwind CSS may be loaded from a CDN.
- Project state lives in a base64-encoded string in the URL, so a plan can be bookmarked, saved, and shared by copying the link.

## Constraints

- **Single page.** All UI, logic, and simulation code live in one `.html` file.
- **No compilation.** Do not use Svelte, Vite, bundlers, TypeScript compilation, or similar. Vanilla JavaScript in `<script>` tags is the implementation.
- **Styling.** Tailwind via CDN is acceptable. Custom CSS in the same file is also fine if needed.
- **Persistence / sharing.** Encode project parameters as a base64url string in the URL **hash**. Loading the page with that URL restores the project. Changing parameters updates the URL so the current state is always shareable.

## Overview

Someone loads the HTML file. They add tasks one by one. For each task they pick a **mode** (likely case) and a **P90** (90th percentile) from Fibonacci hour buttons.

Mathematically, the likely case is the **mode** of the log-normal distribution.

The Monte Carlo simulator then runs many simulations. In each run it samples a duration for every task and **adds those times together**. It draws a histogram of the simulated totals, with vertical lines for the **mode** and **P90** of the resulting distribution.

## Features

- [x] **Task rows.** Add and delete task rows. Delete is a trash icon. Tab order is name → name → Add (estimate chips and delete are not tab stops).
- [x] **Estimates.** Each task has a **Mode** column (likely case) and a **P90** column. Hours for now; a time-unit control comes later.
  - Mode buttons: **1, 2, 3, 5, 8, 13**.
  - P90 buttons: **1, 2, 3, 5, 8, 13, 21**.
  - Columns start **unset**. Clicking the selected chip clears that column.
  - The chip you click wins: a Mode above P90 **raises P90**; a P90 below Mode **pulls Mode down** to the highest legal Fibonacci at or under that P90.
  - Layout: three bands — wide name field, Mode + P90 as one estimate band (equal column width, extra gap between groups), trash in its own action column. Headers use a real P<sub>90</sub> subscript.
- [x] **URL state.** Encode task names, mode, and P90 into the URL (base64url hash, payload `v: 2`). Unset values are `null`. Older name-only hashes still load.
- [x] **URL usage.** Show encoded payload size under the task list (`1,700 bytes`). Remaining headroom and a **2k** / **8k** warn target still TBD.
- [x] **Output.** Monte Carlo histogram of total hours (Chart.js via CDN). Slider for run count (**500–10,000**, default **10,000**). **Re-simulate** draws a new sample. Hours in and out for now.
  - Incomplete estimates: do not invent values. Ignore fully blank rows. If any real task is missing Mode or P₉₀, hide the chart, list what is missing, and disable Re-simulate.
- **Time units.** Choose hours, days, or weeks for inputs and outputs. Underlying calculations convert everything to hours.
- **Calendar / availability.** Account for weekends, holidays, vacation, and sick time. There are several ways to model this; details TBD — come back to it. Related: a **project start date** (until then, assume the plan starts today).
- **Critical-chain planning (later).** Plan the schedule against the modal case, and treat the remaining time (e.g. to P90) as a buffer. Details TBD.
- **Actuals (later).** Record actual durations so critical-chain methods can track buffer consumption against the plan.

## Presentation Features

- [x] **Base view.** Show the histogram of simulated totals (mode and P90 lines as described above). Hours on the axis for now.
- **Calendar overlay.** If weekends and holidays are enabled, show those on the chart.
- **Start date.** For now, assume the plan starts today. Later, capture a start date (see calendar / availability).
- **Waterfall (later).** A “waterfall” plot: a series of histograms on the same chart (one per task / cumulative stage).

## Technical notes

- **Charting.** Start with [Chart.js](https://www.chartjs.org/) via CDN. Later, consider dropping it for straight SVG, possibly with [rough.js](https://roughjs.com/) for a hand-drawn look.
- **URL encoding.** No library. `JSON.stringify` the project → UTF-8 (`TextEncoder`) → **base64url** (`btoa`, then `+`/`/` → `-`/`_`, strip `=`). Reverse with `atob` + `TextDecoder`. Use the hash (`#...`), not the query string, so the payload is not sent to a server.
- **URL length.** The hash is still part of the full URL, so the **browser (and anyone you paste the link into) still limits it**. There is no unlimited hash. Rough budgets: **2k** = conservative / old IE / many chat apps; **8k** = fine in modern Chrome/Firefox. The meter’s 2k vs 8k setting is which of those we warn against — not a hard browser guarantee. If we blow the budget later, tighten the payload (or compress) before adding a library.

## Project plan

Single HTML file. Tailwind via CDN. No build step.

1. [x] **Task list.** Add, delete, and name task rows. Done — interaction locked in `agent-prompt.md`.
2. [x] **Encode / recover.** Persist the task list in the URL hash; reload or share the link and the same list comes back.
3. [x] **Estimates.** Mode and P90 Fibonacci hour buttons on each task; persist selections in the hash. Start unset; the clicked chip is the source of truth for snaps.
4. [x] **Output graph.** Run the simulation and show the histogram (mode and P90 lines).

Then come back and iterate: 2k / 8k remaining-budget meter, time units, calendar / start date, waterfall, critical-chain, actuals, and the rest.

_To be continued._

## Write-up notes

Links to maybe cite later:

- [Hofstadter's Law](https://en.wikipedia.org/wiki/Hofstadter%27s_law)
- [Why software projects take longer than you think — a statistical model](https://erikbern.com/2019/04/15/why-software-projects-take-longer-than-you-think-a-statistical-model.html) (Erik Bernhardsson)
- [Task estimation: conquering Hofstadter’s Law](https://thesearesystems.substack.com/p/task-estimation-conquering-hofstadters)




