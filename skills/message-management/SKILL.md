---
name: message-management
description: >
  Playbook for triaging, summarizing, and organizing incoming Telegram messages/group
  chatter so the user doesn't have to read everything themselves. Use this skill when the
  user asks for a summary/digest of recent messages, asks "what did I miss", wants help
  drafting a reply, or wants ongoing conversations tracked and flagged for follow-up.
version: 1.0.0
metadata:
  hermes:
    tags: [telegram, messaging, triage]
    category: assistant
    fallback_for_toolsets: [messaging]
---

# Message Management

## When to Use
- User asks "what did I miss?" / "summarize my messages" / "any important messages today?"
- User wants a recurring daily/weekly digest (pair with the `reminders-scheduling` skill and
  a cron job that attaches this skill).
- User wants help triaging a busy group chat — what's actionable vs. noise.
- User wants a draft reply to a message/thread.

## Procedure

1. **Gather context, bounded**: pull messages from the relevant Telegram chat(s)/session
   history *since the last checkpoint for that chat* (see "Checkpointing" below), never the
   full history. Respect any per-chat scope the user specifies ("just the family group",
   "just DMs"). If no checkpoint exists yet for a chat, cap the initial pull to a sane
   default window (last 24h, or the last ~200 messages, whichever is smaller) rather than
   scanning everything from the beginning of the chat.
2. **Classify each item** into:
   - **Needs a reply / action** — flag explicitly, with a one-line reason why.
   - **FYI / informational** — summarize briefly, no action needed.
   - **Noise** — skip or mention only in aggregate ("12 low-signal messages in Group X").
3. **Summarize concisely**: lead with anything needing action, then a short digest of the
   rest. Avoid re-explaining messages the user already read/replied to.
4. **Offer, don't assume, next steps**: e.g., "Want me to draft a reply to Alice?" rather
   than sending anything on the user's behalf without confirmation.
5. **For recurring digests**: this skill is designed to be attached to a cron job (see
   `reminders-scheduling`), e.g. a daily 8pm "what did I miss today" job delivered to the
   user's Telegram DM.

## Checkpointing (performance-critical)
- After each digest/summary run, record a checkpoint per chat — the timestamp or message ID
  of the last message included — in memory (e.g. `MEMORY.md` or a small per-chat state note).
- The next run for that chat starts from the checkpoint, not from the beginning of history.
  This keeps each run's cost roughly constant regardless of how long the chat has existed,
  instead of growing (and slowing down) linearly with total message volume over time.
- If a checkpoint is missing, corrupted, or clearly stale (e.g., points to a deleted message),
  fall back to the bounded default window (last 24h / ~200 messages) rather than the full
  history, and re-establish a fresh checkpoint.

## Good defaults for this project
- Default scope is "since last checkpoint for this chat" — see Checkpointing above so digests
  don't repeat old content and don't reprocess ever-growing history.
- Prioritize direct messages and @-mentions over general group chatter when flagging
  "needs a reply."
- Keep digests short — a handful of bullet points, not a transcript replay.

## Pitfalls
- Don't send replies or take actions in chats without explicit user confirmation.
- Don't include sensitive message content in logs/cron output beyond what's needed for the
  summary.
- Don't over-summarize a single important message down to nothing — if something is truly
  time-sensitive, say so plainly instead of burying it in a bullet list.

## Verification
- The digest correctly excludes messages already summarized in a prior run (no repeats).
- Anything flagged "needs a reply" is genuinely actionable — spot-check against the raw
  chat when tuning this skill early on.
- Checkpoints are actually advancing after each run (spot-check `MEMORY.md`/state note) and a
  fresh run on a long-lived, high-volume chat completes quickly instead of pulling full history.

