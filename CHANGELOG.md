# Changelog

Version numbers follow [semver](https://semver.org/): `MAJOR.MINOR.PATCH`.

## v1.7.3 — 2026-04-21

- **Include completed tasks in search.** New "Inc. done" checkbox next to the search input. When ticked, search results also include matching done tasks that would otherwise be hidden by the current view's filter. Preference persists per device.

## v1.7.2 — 2026-04-21

- Completed tasks can be reactivated. Clicking the check-circle on a done task un-marks it and returns it to the active list — with a toast and Undo like the "Marked done" action.
- Done rows now also show an explicit **↺ Restore** button beside the delete button, so the affordance is obvious. Most visible on the Completed view where every row is done.

## v1.7.1 — 2026-04-21

- Brand pill text changed from "TO DO" back to **"FOCAL"**, and recoloured to the Brainbox Candy pink (`#ec008c`) instead of amber. App name is now "Brainbox Candy · Focal" in the browser tab.
- Default sort switched from Priority to **Smart** (priority × 2, minus 20 if overdue, minus 8 if due soon). With no tasks dated today, Smart behaves exactly like Priority — so this is a silent upgrade that starts paying off once due dates get added.
- Sort dropdown hidden from the toolbar by default. New **"Show sort options"** toggle added to Settings → Display. When enabled, the dropdown reappears. State persists per device in localStorage.

## v1.7.0 — 2026-04-21

- Rebranded from **Focal** to **Brainbox To Do**. Uses the real Brainbox Candy logo on the auth screen and in the sidebar, bundled locally in the repo (`logo.jpg`) so no external CDN dependency. Page title, auth subtitle, README and all docs updated.

## v1.6.1 — 2026-04-21

- Gantt bars thickened to 34px with 13px text; rows taller to match. Much easier to read.
- Start date inputs (Quick Add and edit panel) now default to today's date on focus when empty — one click fills them in.
- Added **Low** priority view to the sidebar, with live count badge and filter.

## v1.6.0 — 2026-04-21

- New **Projects** view in the sidebar — Gantt timeline for tasks that have both a start and a due date.
- Added `start_date` column to the `todos` table.
- Quick Add modal and task edit panel now have both Start and Due date fields.
- Gantt shows bars coloured by priority, a vertical today marker, and a red outline on overdue project bars. Range auto-fits to your tasks and rounds to whole weeks (Mon–Sun).
- Bars and task labels are both clickable, opening the full edit panel.

## v1.5.0 — 2026-04-19

- Version pill added to the dashboard strip so the live version is always visible at a glance.
- Default sort changed from Smart to **Priority**. Within a priority band, tasks now order by due date (earliest first, no-date last), then by creation order.
- Dropdown now matches the JS default so the visible option and the applied sort can't disagree.

## Pre-versioning (before v1.5.0)

These changes shipped before the version badge was added.

- **2026-04-17** — Added a "Forgot password?" link to the auth screen.
- **2026-04-17** — Undo added for destructive actions (delete, bulk clear).
- **2026-04-17** — Drag-and-drop reordering now uses fractional `sort_order` values so a single row write is needed per drop (was rewriting every task in the list).
- **2026-04-17** — Realtime Supabase subscriptions now apply changes locally instead of refetching all todos on every update. Much snappier.
- **2026-04-15** — Quick Add space dropdown now defaults to **Work** (was **General**).
- **2026-04-09** — Initial push of the Focal app to GitHub.
