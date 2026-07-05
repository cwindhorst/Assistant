---
name: reminders-scheduling
description: >
  Playbook for creating, managing, and delivering scheduled reminders and recurring
  tasks via Hermes' built-in cronjob tool, delivered to the user on Telegram. Use this
  skill whenever the user asks to be reminded of something, wants a recurring check-in,
  or asks what reminders/jobs currently exist.
version: 1.0.0
metadata:
  hermes:
    tags: [cron, telegram, reminders]
    category: assistant
    requires_toolsets: [cronjob]
---

# Reminders & Scheduling

## When to Use
- The user asks to be reminded of something ("remind me to...", "every morning at 8am...").
- The user wants a recurring automated check (e.g., "check the weather every morning and
  tell me if I need an umbrella").
- The user asks what reminders/cron jobs are currently active, or wants to pause/edit/remove
  one.

## Procedure

Use the built-in `cronjob` tool — do not build a separate reminders system.

1. **Parse the request** into a schedule + a task instruction:
   - One-shot: "remind me in 30 minutes to..." → `cronjob(action="create", schedule="30m", ...)`
   - Recurring: "every morning at 8am..." → `cronjob(action="create", schedule="0 8 * * *", ...)`
     (or the natural-language schedule form Hermes accepts, e.g. `"every 2h"`).
2. **Always set delivery back to the originating chat** (Telegram) unless the user says
   otherwise, so reminders show up where they were requested.
3. **Confirm back to the user** in plain language what was scheduled and when it will next
   fire, so there's no ambiguity.
4. **Pin provider/model on the job** if the user has a non-default model preference for
   reminders (jobs snapshot the global default at creation and fail closed if it later
   changes — see Hermes cron docs). For this project, the default is the configured
   xAI/Grok model, so this usually isn't needed.
5. **Listing/editing/pausing/removing**: use `cronjob(action="list"|"pause"|"resume"|
   "update"|"remove", job_id=...)`. Always confirm which job (by name/id) before removing.
6. **Simple reminders need no skill attached** — only attach a skill to the cron job (via
   `skill=` / `skills=`) when the recurring task itself requires domain knowledge (e.g., a
   "check my messages" job might attach the `message-management` skill).

## Good defaults for this project
- Reminders default to delivering to the Telegram chat the request came from.
- Prefer natural, human-readable schedules over raw cron syntax when confirming to the user
  ("every weekday at 8am" rather than reciting `0 8 * * 1-5`), but store/pass whichever form
  the `cronjob` tool accepts.
- When in doubt about ambiguous timing ("remind me later"), ask a clarifying follow-up
  instead of guessing an exact time.

## Pitfalls
- Don't create a job without confirming the schedule and delivery target with the user first
  if the request is ambiguous.
- Don't silently let a job keep running on a stale pinned model after the user changes their
  global default — flag it and offer to update the pin.
- Don't stack duplicate reminders for the same request; check existing jobs first (`cronjob
  action="list"` / `hermes cron list`) if the request might already be covered.

## Verification
- `hermes cron list` (or `cronjob action="list"`) shows the new job with the expected
  schedule and next-run time.
- The reminder actually fires and is delivered to the correct Telegram chat at the expected
  time — spot-check the first occurrence of any new recurring job.

