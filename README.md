# Focal

The Brainbox Candy task manager — a personal task manager with Supabase sync, built as a single-file HTML app.
Live at **https://msqlpro.github.io/todo-app/**.

## Features

**Tasks.** Priority (Urgent / High / Medium / Low / None), status (Not Started / In Progress / Stuck / Done), start date, due date, notes, tags, and group/space. Tasks can be reactivated from the Completed list via a Restore button or by clicking the check-circle.

**Views.** List, Board (kanban by status), Calendar, and Projects (Gantt timeline for tasks with both a start and a due date). Sidebar filters by view (All / Today / Upcoming / Overdue / Projects), priority, status, and custom spaces.

**Sort.** Hidden by default — the app runs Smart sort (priority × 2, minus 20 if overdue, minus 8 if due within 3 days). The sort dropdown can be revealed via Settings → Display → "Show sort options" for Manual, Priority, Due Date, Status, Created, A–Z, and Group sorts. Drag-to-reorder in Manual mode uses fractional `sort_order` values so each drop is a single row write.

**Search.** Full-text across task text and notes. "Inc. done" checkbox next to the search input pulls matching completed tasks into the results, even in views that normally hide them.

**Assignment.** Any task can be assigned to another user via the edit panel. Assigned-to-you tasks appear in your **Inbox** sidebar and interleaved in your main list with a chip showing who assigned it. Tasks you've assigned away appear in your **Assigned** sidebar, removed from your main list. Both parties can edit any field; only the owner can delete. When the assignee marks done, the owner gets a toast alert next time they open the app and a red count badge on the Assigned entry until they view it. Full spec: [docs/assignment.md](docs/assignment.md).

**Other.** Undo for destructive actions (delete, bulk clear, mark done, reactivate). Apple Notes bulk import. Realtime multi-device sync via Supabase with local state updates. Cross-device auth with forgotten-password flow.

## Tech

- **Frontend:** Single `index.html` — vanilla HTML + CSS + JS, no build step. `logo.jpg` bundled alongside for the Brainbox Candy brand.
- **Backend:** Supabase (Postgres + Auth + Realtime). Project `dbhsxzbqkwuisvvvukxq` — shared with the PD Tracker app.
- **Hosting:** GitHub Pages, served directly from `main`.

## Data model

Main table `public.todos` — `id`, `user_id`, `text`, `done`, `priority`, `status`, `start_date`, `due_date`, `notes`, `category`, `group_name`, `tags[]`, `sort_order`, `created_at`, `completed_at`.

Tasks appear in the **Projects** Gantt view only when both `start_date` and `due_date` are set.

## Preferences

Stored per device in `localStorage`:

- `focalFontSize` — small / medium / large
- `showSortToolbar` — reveal the Sort dropdown in the toolbar
- `searchIncDone` — include completed tasks in search results

## Versioning

`APP_VERSION` and `APP_BUILD` constants near the top of the script control the version pill in the dashboard. Bumped on every meaningful change using [semver](https://semver.org/) (patch / minor / major). See [CHANGELOG.md](CHANGELOG.md).

## Feature specs

Larger features are scoped in advance before any code is written. Current specs:

- [docs/assignment.md](docs/assignment.md) — task assignment between users (target v1.8.0)

## Deploying a change

1. Edit `index.html`.
2. Bump `APP_VERSION` and `APP_BUILD`.
3. **Update `version.json` to match the new `APP_VERSION`.** This is what triggers the auto-update banner in open tabs. If you forget this step, running tabs won't know there's a new version.
4. Add an entry to the top of `CHANGELOG.md`.
5. Update this README if features, data model or preferences changed.
6. Commit + push to `main`. GitHub Pages picks up within a minute. Open tabs will see the banner within 5 minutes (or sooner if the user switches away and back to the tab).
