# Focal

A personal task manager with Supabase sync, built as a single-file HTML app.
Live at **https://msqlpro.github.io/todo-app/**.

## Features

- Tasks with priority (Urgent / High / Medium / Low / None), status (Not Started / In Progress / Stuck / Done), start date, due date, notes, and tags.
- Multiple views: **List**, **Board** (kanban by status), **Calendar**, and **Projects** (Gantt timeline for tasks with both a start and a due date).
- Sidebar filters by view (All, Today, Upcoming, Overdue, Projects), priority, status, and custom spaces (e.g. Work, Personal, Home).
- Quick Add modal plus drag-to-reorder in Manual sort mode.
- Sort by Priority (default, with due-date tiebreaker), Smart, Manual, Due Date, Status, Created, A–Z, or Group.
- Realtime multi-device sync via Supabase, with local state updates to avoid full refetches.
- Undo for destructive actions (delete, bulk clear).
- Apple Notes bulk import.
- Cross-device auth with password reset flow.

## Tech

- **Frontend:** Single `index.html` — vanilla HTML + CSS + JS, no build step.
- **Backend:** Supabase (Postgres + Auth + Realtime). Project shared with the PD Tracker app (`dbhsxzbqkwuisvvvukxq`).
- **Hosting:** GitHub Pages, served directly from `main`.

## Data model

Main table `public.todos` — `id`, `user_id`, `text`, `done`, `priority`, `status`, `start_date`, `due_date`, `notes`, `category`, `group_name`, `tags[]`, `sort_order`, `created_at`, `completed_at`.

Tasks appear in the **Projects** view only when both `start_date` and `due_date` are set.

## Versioning

`APP_VERSION` and `APP_BUILD` constants near the top of the script control the version pill in the dashboard. Bumped on every meaningful change using semver (patch / minor / major). See [CHANGELOG.md](CHANGELOG.md).

## Deploying a change

1. Edit `index.html`.
2. Bump `APP_VERSION` and `APP_BUILD`.
3. Add an entry to `CHANGELOG.md`.
4. Commit + push to `main`. GitHub Pages picks up within a minute.
