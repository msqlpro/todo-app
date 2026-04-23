# Changelog

Version numbers follow [semver](https://semver.org/): `MAJOR.MINOR.PATCH`.

## v1.9.9 — 2026-04-23

- Fix: tasks created from the "Add task…" bar inside a space (e.g. Work, Personal) were being saved into the wrong group. The bar passed the group header label to quick-add, which under smart sort was the generic string `"Tasks"`, not the space name — so the new task landed in a group that didn't match the space's filter, and was effectively invisible from the space view. The bar now always passes the current space name when inside a space, regardless of sort mode. Tasks added from a space correctly appear in that space immediately. Repaired one existing task with this problem (`test no priority` → moved from `General` to `Work`).

## v1.9.8 — 2026-04-23

- Update-check reliability improvements. Two issues were making the "new version available" banner inconsistent:
  1. The initial check ran synchronously at script parse time, before the DOM was ready — if the `updateBanner` element wasn't yet in the tree, the version mismatch was silently detected but no banner could be shown. First check now fires 2 seconds after load.
  2. 5-minute polling was too infrequent for a workflow where users stay on Focal all day. Reduced to 60 seconds so a deploy is picked up within a minute of it becoming visible to the browser.
- Visibility-change handler (tab re-focus) unchanged — still fires a check when you return to the tab.

## v1.9.7 — 2026-04-23

- Fix: Board view's **Done ✓** column was always empty because the list-level filter (used by all views) hides completed tasks as "not active workload". Board now re-injects the most recent 20 completed tasks that would match the current view, specifically for the Done column. List view, counts, and other views are unaffected — only the Board adds the done tasks back. Overdue, In Progress, and Stuck views remain done-free since completion isn't meaningful there.

## v1.9.6 — 2026-04-23

- Fix: completed tasks with a due date were still appearing as pips in the **Calendar** view. The calendar's task map was being built from all tasks with a `due_date` regardless of `done` status. Filter now excludes done tasks, matching the behaviour of Today/Upcoming/Overdue and the list view. The Board view already handles this correctly via its dedicated Done column.

## v1.9.5 — 2026-04-22

- Fix: same empty-state problem as v1.9.4 but for **Events**. When an account had no events, the "+ Add Event" button was hidden because the whole topbar returned early in the empty-state branch. The topbar now renders above the empty state, so new users (or any empty Events view) can add their first event without hunting for the button.

## v1.9.4 — 2026-04-22

- **Fix: "Add task…" bar now shows on empty views too.** Previously, when a view had zero tasks, only the "No tasks here ✦" empty state rendered — leaving no way to add a task except the small **+ Add Task** button in the top-right. Now the familiar **Add task…** bar appears above the empty-state message, consistent with every non-empty view. Clicking it opens the quick-add with the space's name pre-selected as the group (or "General" if outside a space).

## v1.9.3 — 2026-04-22

- Fix: the v1.9.2 update banner was using `position:fixed` and overlapping the dashboard cards at the top of the page. Moved the banner inside the main content column as a regular in-flow block — it now pushes the topbar and content down naturally when visible, and scrolls away as you scroll through tasks. No more occlusion.

## v1.9.2 — 2026-04-22

- **New: auto-update banner.** When a new version is deployed, open tabs now show a pink banner at the top reading *"A new version is available — click to refresh"*. Clicking it force-reloads with a cache-buster to fetch the latest build. Eliminates the need for users to know about Cmd+Shift+R when a fix goes live.
- Check runs on page load, every 5 minutes, and when the tab regains visibility (e.g. user returns from another app). Silent when offline.
- Compares semver version strings properly — `1.10.0` is correctly recognised as newer than `1.9.9`.
- New `version.json` file at the repo root is the source of truth; must be updated alongside each `APP_VERSION` bump going forward.

## v1.9.1 — 2026-04-22

- **New: Assignment history.** Every task now has a server-side audit trail of assignment events — created, assigned, reassigned, unassigned, handed back. Kept forever.
- **Task panel:** new History section below Notes showing that task's assignment trail, newest first. "2h ago · Mark reassigned from Emma to Nicky" style.
- **Settings → Activity log:** global feed of the 100 most recent assignment events you have access to. Each row shows when, who did what, and the task title. Refresh button to reload.
- **Realtime fix:** the `todos` table was never added to the `supabase_realtime` publication, so the app's realtime subscription never received any events — every change required a page reload to see. Added the table to the publication. Changes now propagate to other tabs/users within ~1–2 seconds.
- **DB:** new `public.todo_history` table with RLS (viewers see rows where they were actor/from/to, or where they own/are-assigned-to the current task). An AFTER INSERT/UPDATE trigger captures events automatically — no client-side logging means events can't be faked or missed.

## v1.9.0 — 2026-04-22

- **New: "Back from X" indicator.** When an assignee hands a task back to its owner (by picking the owner from the dropdown, which the spec treats as unassigning), the owner now sees a subtle amber chip on the task row reading **"Back from Nicky"** — so they know the work has returned to their plate rather than the task silently rejoining their main list. The chip has a ✕ for instant dismissal; opening the task panel also clears it automatically.
- **Schema:** new `handed_back_by uuid` column on `public.todos` (nullable, references `auth.users`, ON DELETE SET NULL). Partial index `idx_todos_handed_back_by` on non-null values.
- **RPC:** `reassign_todo()` updated — when the current assignee sets `assigned_to = NULL` on a task they don't own, it stamps `handed_back_by = caller`. Any other reassignment scenario clears the stamp, so the indicator is fresh and always reflects the most recent handback.
- Chip auto-clears on: owner opening the task panel, owner clicking the ✕, subsequent reassignment of the task.

## v1.8.18 — 2026-04-22

- Fix: "Inc. done" search checkbox was leaking completed tasks from unrelated views. E.g. searching "thea" in the Assigned view with Inc. done ticked would re-add every done Thea-related task in the whole database — even personal tasks that had nothing to do with delegation. The re-add step now respects the current view's filter (owner, priority, group, etc.) so only completed items that genuinely belong to the view appear.

## v1.8.17 — 2026-04-22

- Fix: after signing out, the Sign in button on the auth screen was stuck in a disabled "…" state and unresponsive. Cause: the submit handler showed the "…" label during sign-in, then `onAuthStateChange` swapped to the app shell before the button reset fired. On subsequent sign-out the auth screen came back with that stale state. `showAuth()` now resets the button, clears the password field, and hides any lingering error message.

## v1.8.16 — 2026-04-22

- Fixed the sign-out flow. Clicking the user row in the bottom-left sidebar now opens a dropdown menu with **Sign out**, replacing the tiny near-invisible ↪ icon that was only visible on hover. A confirm dialog prevents accidental signouts. Click the row again (or anywhere outside) to close the menu.

## v1.8.15 — 2026-04-22

- The **Assigned** view now excludes completed tasks. Previously it showed everything you'd ever assigned to someone — including long-finished items. The view is for current workload with others; completed assignments still appear in the Completed sidebar view if you need them.

## v1.8.14 — 2026-04-22

- Assignment chips reworded for clarity. `→ Nicky` becomes **Assigned to Nicky**, `← Mark` becomes **From Mark**. Same colours, same click behaviour, clearer meaning at a glance — no arrows to decode.
- Added a subtle **"Signed in as [name]"** pill to the topbar (next to the view toggle). Small, grey, with a green dot. Solves the "which account am I on?" question for anyone who's got two tabs open for testing, without disrupting the main layout.

## v1.8.13 — 2026-04-22

- **All Tasks** sidebar badge and dashboard **Total** card now count only active (not-done) tasks. The Completed view has its own count — there's no need for finished items to inflate the "All Tasks" number. Matches the behaviour of every other sidebar badge (Today, Upcoming, priorities, etc). The Settings → Account row "Total tasks" still shows the lifetime count, since that panel is summarising history rather than workload.
- Overall Progress percentage is unchanged — still computes done ÷ lifetime total.

## v1.8.12 — 2026-04-22

- Fix: assignee-to-owner reassignment via new SECURITY DEFINER RPC `reassign_todo(p_id, p_new_assigned_to)`. The regular UPDATE path was being blocked by trigger + RLS interplay — when Nicky set `assigned_to=NULL` to pass a task back, Postgres rejected it with *"new row violates row-level security policy"* even though the policy should have permitted the write. Worked around by routing all assignee-driven assignment changes through a database function that bypasses RLS and performs its own permission check (must be current owner or current assignee). Owner-driven changes still use the regular update path.

## v1.8.11 — 2026-04-22

- Fix: silent save failure when an assignee picked the owner from the Assigned-to dropdown. After the update set `assigned_to=null`, the assignee no longer had RLS read access, so the chained `.select().single()` failed with PGRST116 and the save appeared broken. Dropped the select-back; merged the patch into local state directly and removed the task from view if it's no longer RLS-visible to the current user.

## v1.8.10 — 2026-04-22

- Inbox and Assigned sidebar entries auto-hide when empty and reappear as soon as they have content. Cleaner sidebar for users who rarely receive assignments (or rarely make them). The entry stays visible if you're actively viewing it even if the last item gets cleared mid-session, so it doesn't disappear under you.

## v1.8.9 — 2026-04-22

- Hide the **"Add date"** placeholder chip from task rows. The due-date chip now only renders when a date is actually set, so tasks with real deadlines stand out and the list looks less busy. Dates can still be added via the edit panel's **Due Date** field.

## v1.8.8 — 2026-04-22

- Move **Save Changes** and **Delete** buttons from the bottom of the task edit panel to the top header, next to Close. Avoids having to scroll down on long tasks to save. Buttons are slightly smaller in the header to keep the layout tidy.
- Removed the now-empty panel footer to reclaim vertical space for notes.

## v1.8.7 — 2026-04-22

- Remove "Create account" tab from the auth screen. Focal is now a staff-only app — Supabase signups have been disabled project-wide, so the tab would only ever produce a confusing error. Auth screen simplified to sign-in only.
- Add a small footnote below the Forgot password link: *"Focal is for Brainbox Candy staff. Need access? Ask Mark."* — so anyone who stumbles on the URL knows where to go.
- Dropped the orphaned `isSignUp` state and both tab click handlers.

## v1.8.6 — 2026-04-22

- Drop the "unseen done" badge format on the Assigned sidebar — it was fiddly and not adding enough value over a plain count. Badge now shows just the number of active tasks currently assigned to others, like every other sidebar counter.

## v1.8.5 — 2026-04-22

- Fix: `updateTodo` now surfaces Supabase errors instead of silently swallowing them. Previously, if any row write was rejected (RLS, trigger, constraint), the UI would update local state as if it succeeded — so the app could drift out of sync with the database without any feedback. Failed writes now show a toast and trigger a full reload so local state matches reality.
- Task writes now use `.select().single()` to return the DB row, so trigger-computed fields like `assigned_at` are reflected locally immediately.
- Data repair: cleared a stuck assignment on the "Meeting with Thea" task where Nicky's reassignment back to Mark had silently failed under the pre-fix code path.

## v1.8.4 — 2026-04-22

- Fix: the Assigned sidebar badge showed `0` even when tasks were assigned out, because v1.8.0 counted only unseen completions (red notifications) rather than total assignments. Now shows the total count of active tasks assigned to others (e.g. `2` when you have two tasks with Nicky). When there are unseen completions on top, the label becomes e.g. `2 · 1 done` and turns red until you've acknowledged them.

## v1.8.3 — 2026-04-22

- Fix: assignee couldn't pass a task back to the owner by picking them in the Assign dropdown. The picker had been excluding the task owner in all contexts; now it excludes whoever is currently viewing. The owner appears in the list with an "(owner)" suffix so the direction is clear.
- Per-spec rule 4, picking the owner from the dropdown is treated as unassigning — `assigned_to` is set to NULL and the task returns to the owner's main list. Both the edit panel and Quick Add enforce this.
- Hint text reworded for assignees: "Pick their name to pass it back, or pick someone else to reassign".
- "Not assigned" option hidden for non-owners (it's only meaningful for the owner — for assignees, "not assigned" and "assigned to the owner" mean the same thing).

## v1.8.2 — 2026-04-22

- Fix: Add Task button in the Quick Add modal unresponsive. The new Assign-to row (from v1.8.1) was missing its closing `</div>`, which swallowed the actions row and broke the button's click target. Closed the tag properly.

## v1.8.1 — 2026-04-22

- Quick Add modal gains an **Assign to** picker on its own row, so a task can be assigned at creation instead of needing a round-trip through the edit panel. Default is "Me (keep)" — assignment is explicit opt-in. Toast confirms: *"Task added and assigned to Nicky ✓"* when used.

## v1.8.0 — 2026-04-21

Task assignment between users. Full spec at [docs/assignment.md](docs/assignment.md).

- **Database** — three new columns (`assigned_to`, `assigned_at`, `assignee_seen`), two updated RLS policies (SELECT and UPDATE now include assignees), a BEFORE UPDATE trigger that auto-manages `assigned_at` on reassignment and flips `assignee_seen` on completion / reactivation.
- **Profiles** — added `email` column to existing `public.profiles` table, signup trigger keeps it in sync. Used by the assign picker to show friendly names.
- **Assign picker** — new "Assigned to" field in the task edit panel, lists all other users with "— Not assigned" as the clear option. Contextual hint line tells you who the task is with.
- **Sidebar** — two new entries under Views: **Inbox** (tasks assigned to you) with a pink count badge, and **Assigned** (tasks you've assigned to others) with a red unseen-completion badge. Both forced to list display.
- **Row chip** — tasks show a small `→ Nicky` or `← Mark` chip indicating assignment direction. Completed-but-unseen tasks get a red "done" chip until the owner opens or dismisses them.
- **Main views** — All Tasks, Today, Upcoming, priority views, status views, and space views now hide tasks you've assigned away. Assigned-to-you tasks appear in them in priority order with the chip.
- **Completion alerts** — when the assignee marks a task done, the owner gets a toast per task next time they load the app: *"Nicky completed: '…'"*. Badge on the Assigned sidebar shows unseen count. Clears when the owner opens the edit panel.

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
