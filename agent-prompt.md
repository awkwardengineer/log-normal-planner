# Agent prompt — rebuild the task list

You are rebuilding **Log-Normal Planner** from scratch. This prompt is the only spec you need. Recreate `index.html` to match what is specified here. Do not invent a different UI. Do not add a toolchain, framework, or extra source files.

## Product

A single-file project planner that will use **log-normal Monte Carlo** simulation to estimate timelines from uncertain task estimates.

Someone loads one HTML file. They add tasks one by one. Later, each task will have a **likely case** and a **P90** estimate (sliders). The likely case is the **mode** of a log-normal. The simulator will sample a duration for every task, **add those times together**, and draw a histogram of the totals with vertical lines for the **mode** and **P90**.

Project state will eventually live in a **base64url** string in the URL **hash** (not the query string) so a plan can be bookmarked and shared. Encoding: `JSON.stringify` → UTF-8 (`TextEncoder`) → base64url (`btoa`, then `+`/`/` → `-`/`_`, strip `=`). Reverse with `atob` + `TextDecoder`. No encoding library. The hash is still part of the full URL, so browsers and chat apps still limit length (rough warn targets: 2k conservative, 8k modern Chrome/Firefox).

Other later work (do not build now): URL budget meter (2k vs 8k); time units (hours / days / weeks, calculate in hours); weekends, holidays, vacation, sick time, and a start date (assume today until that exists); Chart.js via CDN for the histogram, maybe straight SVG + rough.js later; calendar overlay on the chart; a waterfall of histograms; critical-chain (schedule to the mode, leftover time as buffer); recording actuals.

## This slice only

Build the **named task list**. Estimates, hash persistence, simulation, and the chart come after. Today, only add / name / delete tasks, with the interaction and visuals below.

## Constraints

- Single file: `index.html`. Markup, styles, and all application JavaScript in that file.
- No build, no bundler, no Svelte/React/Vue, no TypeScript compile. Vanilla JS in `<script>` tags.
- Tailwind from `https://cdn.tailwindcss.com`. Custom CSS in the same file is fine if needed.
- Open the file (or a static server) in a browser; it works.

## What to ship

A page titled **Log-Normal Planner** with subtitle: *Add tasks, then name them. Estimates come next.*

A white card (`max-w-2xl`, slate page background) with:

1. A **Tasks** heading only. **No Add button in the header.** A header Add was tried and rejected: it is the loudest control and sits away from the work.
2. A list of task rows. Each row: text input (placeholder `Task name`) + **Delete**.
3. When the list is **empty:** a dashed, clickable empty state: **+ Add a task to get started**. Same action as Add.
4. When the list has **any tasks:** hide the empty state. Show a footer with one **primary** Add task button (dark fill, left-aligned — not a full-width ghost row).
5. After a delete: a toast under the card (not inside it) until undo expires or is used.

Treat the list as an **editor**, not a form. One Add, at the bottom of the work.

## Data

```js
tasks = [{ id: number, name: string }]
nextId = 1
undo = null | { task, index, timeoutId }
```

`id` is a monotonically increasing integer, stable for the session. Do not persist yet.

## Interaction contract (must match)

### Add

- Empty-state click and footer **Add task** both append a new `{ name: "" }` and **focus its input**.
- **Enter** in a name field:
  - If the name is non-empty: insert a new blank task **immediately below** this row and focus it.
  - If the name is empty and this is the **last** row: do nothing (do not spawn another blank).
  - If the name is empty and this is **not** last: focus the next row’s input.

### Delete

- Row **Delete** removes that task immediately. **No confirm dialog.**
- **Backspace** in a name field that is already `""` deletes that row.
- After delete, restore focus: **previous** name if any, else **next** name, else the Add control (empty state if the list is now empty, else the footer Add button). Never leave focus on `document.body`.

### Undo

- One level only. A new delete replaces the previous undo.
- Toast for **5 seconds**, then it disappears.
- Copy: `Deleted “{name}”.` or `Deleted untitled task.`
- **Undo** restores that task at its old index and focuses its name.
- Toast is `role="status"` + `aria-live="polite"`.
- **Show/hide:** do **not** put Tailwind `hidden` and `flex` on the element at the same time and expect `hidden` to win. `flex` overrides `[hidden]` / `display: none`. Toggle: `hidden` when idle, `flex` when armed. Dark bar (`bg-slate-900`), light text, white **Undo** button.

### Escape

- Escape while focus is inside `main` **blurs** the active control (name, Delete, Add, Undo). That is “done editing,” not delete.

### Delete button a11y

- Visible label: `Delete`.
- `aria-label`: `Delete {name}` or `Delete untitled task`.
- Hit target at least 32px (`h-8`). Chip at rest; red on hover.

## Visual system (must match)

Page: `bg-slate-50`, `text-slate-900`. Card: white, `rounded-xl`, light border, light shadow.

| Role | Treatment |
| --- | --- |
| Primary (Add, when list is non-empty) | `bg-slate-900` text white; hover `bg-slate-700`; **white** focus ring (`focus-visible:outline-white` + offset). Dark-on-dark rings are invisible. |
| Empty state | Dashed border, `+ Add a task to get started`, looks like a drop target, not gray prose. |
| Delete rest | Light chip (`bg-slate-100 text-slate-500`). Not ghost text — ghost looks disabled. |
| Delete hover | `bg-red-50 text-red-600`. |
| Other focus rings | Slate-900 outline, 2px, offset 2px (`:focus-visible`). |
| Name field focus | Visible border / outline; this is the main editing surface. |

Do not use faint `text-slate-400` for a real action.

## Implementation notes

- Vanilla JS. Re-render the `<ul>` from `tasks` on add/delete (and refresh undo chrome). After `replaceChildren`, **re-apply focus** from render options (`focusTaskId` / `focusAdd`).
- Name edits update `task.name` on `input` (no persist).
- `autocomplete="off"` on name fields.
- Keep the file small and readable. No comments that restate the code.

## Out of scope (do not add while rebuilding this slice)

URL hash / base64url, URL budget meter, likely/P90 sliders, time units, calendar, Chart.js, Monte Carlo, waterfall, critical-chain, actuals.

Known follow-ups **not** in the current app (do not implement unless asked):

- ⌘Z / Ctrl+Z for undo
- ↑ / ↓ between name fields
- Take Delete out of Tab order (`tabindex="-1"`); Tab = name → name → Add
- Softer Backspace: only delete the row if the field was already empty **when it focused**, not on the keystroke that cleared the last character
- If Add would create a second trailing blank, focus the existing blank instead

## Acceptance

- Empty page: dashed empty state only; no header Add; no undo toast visible.
- Add from empty state → focused blank name. Enter after typing a name → new row below, focused.
- Enter on a trailing blank does not add another blank.
- Delete and empty-Backspace remove the row; focus stays in the list; toast names the task; Undo restores it; toast gone after 5s or after Undo.
- Escape blurs.
- One primary Add, bottom-left of the card, only when there are tasks.

Rebuild `index.html` to satisfy this prompt. Do not jump ahead of this slice.
