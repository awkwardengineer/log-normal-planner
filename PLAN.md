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

Someone loads the HTML file. They add tasks one by one. For each task they use a slider to provide a **likely case** and a **P90 case** estimate.

Mathematically, the likely case corresponds to the **mode** of the log-normal distribution.

The Monte Carlo simulator then runs many simulations. In each run it samples a duration for every task and **adds those times together**. It draws a histogram of the simulated totals, with vertical lines for the **mode** and **P90** of the resulting distribution.

## Features

- **Task rows.** Add and delete task rows.
- **Time bounds.** Set a low-bound time and a high-bound time. The low bound can never exceed the high bound, and vice versa.
- **URL state.** Encode task names and time estimates into the URL (base64) so the project can be saved and shared.
- **URL budget.** Show how much encoded “memory” (remaining characters) is still available. Meter can target a **2k** or **8k** limit.
- **Time units.** Choose hours, days, or weeks for inputs and outputs. Underlying calculations convert everything to hours.
- **Calendar / availability.** Account for weekends, holidays, vacation, and sick time. There are several ways to model this; details TBD — come back to it. Related: a **project start date** (until then, assume the plan starts today).
- **Critical-chain planning (later).** Plan the schedule against the modal case, and treat the remaining time (e.g. to P90) as a buffer. Details TBD.
- **Actuals (later).** Record actual durations so critical-chain methods can track buffer consumption against the plan.

## Presentation Features

- **Base view.** Show the histogram of simulated totals (mode and P90 lines as described above).
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
2. **Encode / recover.** Persist the task list in the URL hash; reload or share the link and the same list comes back.
3. **Estimates.** Add high-bound and low-bound estimates on each task (low bound cannot exceed high bound, and vice versa).
4. **Output graph.** Run the simulation and show the histogram (mode and P90 lines).

Then come back and iterate: URL budget meter, time units, calendar / start date, waterfall, critical-chain, actuals, and the rest.

_To be continued._



