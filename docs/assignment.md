# Task assignment — feature specification

**Status:** shipped in v1.8.0 (2026-04-21)
**Target version:** v1.8.0

Every task may optionally be assigned from its owner to another user. Assigned tasks are visible and editable by both parties, show in dedicated sidebar sections, and trigger a completion alert back to the owner.

This document is the contract. Anything not stated here is out of scope for v1.

---

## Ownership and visibility

- Every task has one **owner** — `user_id`, set at creation, never changes.
- Every task may have one **current assignee** — `assigned_to`, nullable, can change over time.
- An unassigned task (`assigned_to` IS NULL) is visible only to its owner. Behaviour is unchanged from pre-v1.8.
- An assigned task is visible to both the owner and the current assignee. Both can edit every field (text, priority, status, dates, notes, tags, spaces, sort order).
- **Only the owner can delete.** An assignee can reassign a task away from themselves but never delete it.
- The owner always retains visibility on their tasks, even after assigning.

### RLS policies

| Action | Condition |
| --- | --- |
| SELECT | `auth.uid() = user_id` OR `auth.uid() = assigned_to` |
| INSERT | `auth.uid() = user_id` |
| UPDATE | `auth.uid() = user_id` OR `auth.uid() = assigned_to` |
| DELETE | `auth.uid() = user_id` |

### Reassignment RPC

Regular UPDATEs work when the user retains RLS access after the write. But when an assignee hands a task back to the owner (setting `assigned_to = NULL`), the UPDATE strips the assignee's own access mid-transaction — Postgres rejects this as *"new row violates row-level security policy"* even without a WITH CHECK clause. Workaround: a `SECURITY DEFINER` function `public.reassign_todo(p_id, p_new_assigned_to)` that bypasses RLS and performs its own permission check (caller must be the current owner or current assignee). The frontend routes all non-owner assignment changes through this RPC; owner-driven changes and all other field updates continue to use the regular UPDATE path.

## Data model changes

Columns on `public.todos`:

| Column | Type | Default | Notes |
| --- | --- | --- | --- |
| `assigned_to` | `uuid` | NULL | References `auth.users(id)`. NULL = unassigned. |
| `assigned_at` | `timestamptz` | NULL | Set when `assigned_to` becomes non-NULL. NULL when unassigned. |
| `assignee_seen` | `boolean` | `true` | Set to `false` when the assignee marks the task done. Cleared to `true` when the owner views or dismisses it. |
| `handed_back_by` | `uuid` | NULL | Set by `reassign_todo()` when a non-owner assignee reassigns their task to NULL (handing it back to the owner). References `auth.users(id)` with `ON DELETE SET NULL`. Cleared by owner viewing the task, owner clicking the ✕ on the chip, or any subsequent reassignment. |

Everything else unchanged. Existing tasks get `assigned_to=NULL`, `assigned_at=NULL`, `assignee_seen=true`, `handed_back_by=NULL` — i.e. behave exactly as they do today.

## Assigner's experience

When a user (the owner) assigns a task:

- The task **leaves** their main-list views — All Tasks, Today, Upcoming, priority views (Urgent/High/Medium/Low), status views, and space views (Personal/Work/etc.) all exclude tasks the current user has assigned away.
- It appears in a new **"Assigned"** sidebar section under the Views heading. That section lists every task the current user owns where `assigned_to` is not NULL and not themselves.
- Rows in the Assigned section show an "→ Nicky" chip indicating the current assignee.
- Sort within the Assigned section follows the same global sort as other views.
- The owner can edit any field, unassign (sets `assigned_to=NULL`, returns the task to main list naturally), or reassign to anyone else.
- Completed-by-assignee tasks remain in the Assigned section until the owner dismisses the notification (see Completion & alerts below). They also appear in Completed.

## Assignee's experience

When a user is assigned a task:

- It appears in **two places simultaneously**:
  1. **Inbox** — a new sidebar section listing every task currently assigned to them, newest unseen first, grouped visually by assigner.
  2. **Main list views** — All Tasks, Today, Upcoming, priority views, status views, space views. Assigned tasks are interleaved with the assignee's own tasks in priority order, each with an "← Mark" chip indicating who assigned it.
- Rationale for appearing in both: an Inbox is reassuring ("nothing will be missed"), but integrated priority placement ensures the assignee sees assigned work in context with their own workload.
- The assignee can edit every field (priority, due date, notes, status, etc.) exactly like their own tasks.
- The assignee can reassign to anyone else (including back to the original owner).
- The assignee cannot delete.

## Reassignment

- Any user who can see a task (owner or current assignee) can change `assigned_to`.
- Reassignment overwrites — no chain or history preserved in v1.
- When reassigned, the previous assignee loses visibility immediately (unless they are the owner).
- **Reassigning to the owner is the same as unassigning.** Sets `assigned_to=NULL`. The task returns to the owner's main-list views naturally.
- On reassignment, `assigned_at` updates to the new moment.

## Completion and alerts

- When the current assignee marks a task done (`done=true`, `completed_at` set), the database trigger also sets `assignee_seen=false` **if the assignee is not the owner**. Owner self-completing their own task never triggers an alert.
- Next time the **owner** opens the app, any of their tasks with `assignee_seen=false` produce:
  - One toast per task: *"Nicky completed: 'Reorder envelopes'"*
  - A red count badge on the Assigned sidebar entry showing total unseen completions
- The badge and unseen flag clear when the owner either:
  - Opens the task's edit panel, OR
  - Clicks an explicit "Dismiss" action on the row in the Assigned section
- **Reactivation resets the cycle.** If the owner un-dones an assigned task that was completed, `assignee_seen` is set back to `true`. If the assignee later completes again, a fresh alert fires.
- If the **current assignee is not the owner** and the assignee reactivates their own completion, same reset applies.
- The alert fires **only to the owner** — not to any intermediate assigners in a reassignment chain (which doesn't exist in v1 anyway).

## User picker

- The Assign dropdown shows every user in `auth.users` except the currently logged-in user.
- Display name = the part of the email before `@`, title-cased. Tooltip shows the full email.
- No team/organisation concept yet. Fine while the list is small (currently Mark + Nicky). Revisit if the picker becomes unwieldy.

## Confirmed rules (from business-rules finalisation)

1. Owner always retains visibility. ✓
2. Only owner can delete. ✓
3. Completion alert fires only to the owner. ✓
4. Reassigning to owner = unassigning (`assigned_to=NULL`). ✓
5. Alert clears on view or explicit dismiss. ✓
6. Badge counts unseen completions — one per task, accumulating. ✓

## Out of scope for v1

- Email notifications — in-app toast only.
- Real-time push from Supabase realtime — works out of the box since it's the same `todos` table, but no special handling (no animation, no "Nicky just assigned you a task" banner).
- Comments or discussion thread on a task.
- Assignment history / chain visibility.
- Assigning to multiple people at once.
- Teams, roles, permissions beyond the owner/assignee model.
- Bulk assign.
- "Request changes" / rejection flow — assignee either does the task or reassigns it.

## Open questions for future versions

- Should the Inbox also show historical assignments (things completed by the assignee themselves)?
- Should owners get an "Assignments activity" log in Settings?
- Is there a need to see "what Nicky is working on right now" for project management — a dashboard view across users?

These are parked, not decisions.
