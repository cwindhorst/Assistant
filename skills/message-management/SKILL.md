---
name: message-management
description: >
  Playbook for triaging, summarizing, and organizing incoming Telegram messages/group
  chatter so the user doesn't have to read everything themselves. Use this skill when the
  user asks for a summary/digest of recent messages, asks "what did I miss", wants help
  drafting a reply, or wants ongoing conversations tracked and flagged for follow-up.
---

# Message Management

## When to use this skill
- User asks "what did I miss?" / "summarize my messages" / "any important messages today?"
- User wants a recurring daily/weekly digest (pair with the `reminders-scheduling` skill and
  a cron job that attaches this skill).
- User wants help triaging a busy group chat — what's actionable vs. noise.
- User wants a draft reply to a message/thread.

## How to do it

1. **Gather context**: pull recent messages from the relevant Telegram chat(s)/session
   history. Respect any per-chat scope the user specifies ("just the family group", "just
   DMs").
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

## Good defaults for this project
- Default scope is "since last time I asked" or "since last digest" — track what's already
  been summarized so digests don't repeat old content.
- Prioritize direct messages and @-mentions over general group chatter when flagging
  "needs a reply."
- Keep digests short — a handful of bullet points, not a transcript replay.

## Things to avoid
- Don't send replies or take actions in chats without explicit user confirmation.
- Don't include sensitive message content in logs/cron output beyond what's needed for the
  summary.
- Don't over-summarize a single important message down to nothing — if something is truly
  time-sensitive, say so plainly instead of burying it in a bullet list.
