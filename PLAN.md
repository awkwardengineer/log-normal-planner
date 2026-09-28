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

Someone loads the HTML file. They add tasks one by one. For each task they pick a **mode** (likely case) and a **P90** (90th percentile) from Fibonacci **day** buttons.

Mathematically, the likely case is the **mode** of the log-normal distribution.

The Monte Carlo simulator then runs many simulations. In each run it samples a duration for every task and **adds those times together**. It draws a histogram of the simulated totals, with vertical lines for the **mode** and **P90** of the resulting distribution.

## Features

- [x] **Task rows.** Add and delete task rows. Delete is a trash icon. Tab order is name → name → Add (estimate chips and delete are not tab stops).
- [x] **Estimates.** Each task has a **Mode** column (likely case) and a **P90** column. Phase 2: chips are **days**. Hours / days / weeks switching is Phase 3.
  - Mode buttons: **1, 2, 3, 5, 8, 13**.
  - P90 buttons: **1, 2, 3, 5, 8, 13, 21**.
  - Columns start **unset**. Clicking the selected chip clears that column.
  - The chip you click wins: a Mode above P90 **raises P90**; a P90 below Mode **pulls Mode down** to the highest legal Fibonacci at or under that P90.
  - Layout: three bands — wide name field, Mode + P90 as one estimate band (equal column width, extra gap between groups), trash in its own action column. Headers use a real P<sub>90</sub> subscript.
- [x] **URL state.** Encode task names, mode, and P90 into the URL (base64url hash, payload `v: 2`). Unset values are `null`. Older name-only hashes still load. Phase 2 also stores **start date**, **weekends**, **federal holidays**, and **winter break** in that same hash (no version bump). Old hour-based links are thrown out.
- [x] **URL usage.** Show encoded payload size under the task list (`1,700 bytes`). Remaining headroom and a **2k** / **8k** warn target still TBD.
- [x] **Output.** Monte Carlo histogram of total duration (Chart.js via CDN). Slider for run count (**500–10,000**, default **10,000**). **Re-simulate** draws a new sample. Phase 2: display in **elapsed calendar days** plus calendar dates.
  - Incomplete estimates: do not invent values. Ignore fully blank rows. If any real task is missing Mode or P₉₀, hide the chart, list what is missing, and disable Re-simulate.
- [x] **Time units (Phase 2).** UI is **days** (same Fibonacci chips). Simulation stays in **hours** (`1 day = 8 hours`). Hours / days / weeks switching is Phase 3 — no unit field in the hash yet.
- [x] **Calendar / availability (Phase 2).** Start-date picker, skip weekends, federal holidays, winter break. Defaults: **start date = today**, **weekends on**, **federal holidays off**, **winter break off**. All live in the URL hash. Vacation and sick time still later.
- **Critical chain.** Schedule against the best case and pool the safety into one buffer. See [Critical chain](#critical-chain).

## Presentation Features

- [x] **Base view.** Histogram of simulated totals (mode and P90 lines). Phase 2: x-axis is **elapsed calendar days** from the start date, with calendar-date labels. Non-work days are empty bins.
- **Calendar overlay (later).** Extra shading of weekend and holiday bands, if we want more than the empty bins.
- [x] **Start date.** Date picker at the top of the task inputs. Default **today**. Persist in the hash with weekends and holidays.
- [x] **Waterfall.** Small finish-time histograms, one per task: each row is when that task finishes if they run in list order (prefix sums). Same elapsed-calendar axis as the total graph.
- [x] **Schedule (Gantt).** Best-case bars plus a pooled buffer bar. See [Critical chain](#critical-chain).

## Critical chain

Padding every task individually is how estimates rot: the safety is invisible, it gets spent, and it never rolls forward. Critical chain does the opposite. **Schedule the work at the best case, strip the safety out of the tasks, and hold it in one shared buffer at the end.** The buffer is the commitment; the task bars are not promises.

The buffer is **not** the sum of each task's gap to its P₉₀. Risk aggregates: the simulated total at the target percentile is smaller than every task going badly at once. That difference is the whole argument for the method, so the UI says both numbers out loud.

- **Chain.** Tasks run back to back in list order at their **best case** (the low bound). Parallel work and real dependencies are later.
- **Commit level.** The buffer runs from the end of the chain to the simulated total at **P₇₀ or P₉₀** — how aggressive you want the promise to be. Persisted in the hash (`c`).
- **Buffer.** `commit − chain`, clamped at zero. One bar, drawn after the last task.

### Steps

1. [x] **Best case + buffer chart.** Horizontal bars on the shared elapsed-calendar axis: one per task at its best case, then the buffer. Date labels on top, calendar days on the bottom, same as the other graphs. A note under the chart states the chain length, the buffer, the commit date, and what per-task padding would have cost.
2. **Actuals.** Mark a task done (and how long it really took). The chain reflows from there and overruns **eat buffer**; everything downstream slides right.
3. **Buffer burn.** Show how much buffer is gone against how much of the chain is complete — the fever chart. Green / yellow / red zones, so "we're late" becomes a measurement instead of a feeling.
4. **Feeding buffers.** Once tasks can run in parallel, side chains need their own smaller buffers where they join the critical chain.

Open questions: whether the best case should be the mode or something more aggressive (P₅₀); whether re-simulating after actuals should re-fit the remaining tasks from observed error.

## Technical notes

- **Charting.** Start with [Chart.js](https://www.chartjs.org/) via CDN. Later, consider dropping it for straight SVG, possibly with [rough.js](https://roughjs.com/) for a hand-drawn look.
- **URL encoding.** No library. `JSON.stringify` the project → UTF-8 (`TextEncoder`) → **base64url** (`btoa`, then `+`/`/` → `-`/`_`, strip `=`). Reverse with `atob` + `TextDecoder`. Use the hash (`#...`), not the query string, so the payload is not sent to a server.
- **URL length.** The hash is still part of the full URL, so the **browser (and anyone you paste the link into) still limits it**. There is no unlimited hash. Rough budgets: **2k** = conservative / old IE / many chat apps; **8k** = fine in modern Chrome/Firefox. The meter’s 2k vs 8k setting is which of those we warn against — not a hard browser guarantee. If we blow the budget later, tighten the payload (or compress) before adding a library.

## Project plan

Single HTML file. Tailwind via CDN. No build step.

1. [x] **Task list.** Add, delete, and name task rows. Done — interaction locked in `agent-prompt.md`.
2. [x] **Encode / recover.** Persist the task list in the URL hash; reload or share the link and the same list comes back.
3. [x] **Estimates (Phase 1, shipped).** Mode and P90 Fibonacci **hour** buttons on each task; persist selections in the hash. Start unset; the clicked chip is the source of truth for snaps.
4. [x] **Output graph (Phase 1, shipped).** Run the simulation and show the histogram (mode and P90 lines) in hours.
5. [x] **Waterfall.** Finish-time ridge plot (cumulative time to finish each task, list order).

### Phase 2

Still one HTML file. Tailwind via CDN. No build step. The UI speaks **days**. Simulation stays in **hours** (`1 day = 8 hours`).

1. [x] **Days in and out.** Mode / P90 chips are **days** (`1, 2, 3, 5, 8, 13` and P90 `+ 21`). Convert to hours for the log-normal fit and Monte Carlo. **Do not bump the URL payload.** Old hour-based links are thrown out (same `v: 2` shape, new meaning). Units in the hash are Phase 3.
2. [x] **Start date.** Date picker at the top of the task inputs. Default **today**. Persist in the URL hash. Calendar origin for every simulated duration.
3. [x] **Elapsed calendar axis (both graphs).** Plot in **elapsed calendar days** from the start date (this stretches the shape across non-work days). Dual labels: calendar days and the **calendar date**. Non-work days are empty bins.
4. [x] **Weekends.** Checkbox **Skip weekends**. **Default on.** Persist in the URL hash. Saturday and Sunday are non-working; work resumes Monday. A working day is 8 hours.
5. [x] **Holidays.** Two checkboxes, both **default off**, both in the URL hash:
   - **Federal holidays:** US federal holidays (usual weekday observance) plus the day after Thanksgiving.
   - **Winter break:** **December 24 through January 1** inclusive.

Work calendar applies when mapping effort hours onto calendar dates. The Monte Carlo still sums task hours; it does not simulate “Saturday work.”

### Phase 3

1. **Unit switching.** Control to display and enter estimates in **hours, days, or weeks**. Persist the selected unit in the URL. Convert at the edges; simulation stays in hours. This is where we handle units in the payload (Phase 2 does not).

Then come back and iterate: calendar overlay (shade non-work days), 2k / 8k remaining-budget meter, vacation / sick time, the rest of [Critical chain](#critical-chain), and the rest.

_To be continued._

## Write-up notes

Links to maybe cite later:

- [Hofstadter's Law](https://en.wikipedia.org/wiki/Hofstadter%27s_law)
- [Why software projects take longer than you think — a statistical model](https://erikbern.com/2019/04/15/why-software-projects-take-longer-than-you-think-a-statistical-model.html) (Erik Bernhardsson)
- [Task estimation: conquering Hofstadter’s Law](https://thesearesystems.substack.com/p/task-estimation-conquering-hofstadters)




