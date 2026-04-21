# Changelog

Version numbers follow [semver](https://semver.org/): `MAJOR.MINOR.PATCH`.

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
