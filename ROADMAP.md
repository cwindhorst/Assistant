# Roadmap: From "Hermes Agent installed" to "functional personal assistant"

This answers issue #1 — "What will it take to complete this project."

Hermes Agent ([NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)) is
already installed on the Linux machine. Completing this project is about **configuring,
wiring up, and teaching** that agent — not writing new agent code. The work below runs on
the Linux box; this repo tracks the plan and holds reference config/skill files to copy over.

## Phase 0 — Baseline check
- Confirm the install is current: `hermes update` (or re-run the installer).
- Confirm `~/.hermes/` exists with `config.yaml`, `.env`, `SOUL.md`, `skills/`, `cron/`.
- Run `hermes config check` to catch any existing misconfiguration.

## Phase 1 — Model provider (xAI / Grok)
- Get an xAI API key from the xAI console.
- Set `XAI_API_KEY` in `~/.hermes/.env` (secrets never go in `config.yaml`).
- Set the default model: `hermes config set model xai/<grok-model-id>` (check
  `hermes model` for the current supported Grok model ids — these change over time).
- Verify with a plain `hermes chat` session before wiring any platform.
- See [`config/config.yaml.example`](config/config.yaml.example) and
  [`config/.env.example`](config/.env.example) in this repo.

## Phase 2 — Telegram gateway
- Create a bot via **@BotFather** (`/newbot`), grab the bot token.
- Run `hermes gateway setup` (interactive wizard — prompts for the token, writes it to
  `.env`) or wire it by hand using the `platforms.telegram` block in
  [`config/config.yaml.example`](config/config.yaml.example).
- **Turn off privacy mode** in BotFather (`/mybots` → Bot Settings → Group Privacy) if the
  bot needs to see all messages in a group, not just `/commands` — for personal DM use this
  isn't necessary.
- Start the gateway (`hermes gateway`) and confirm you can chat with the bot from your phone.
- Optionally enable `status_indicator` so the bot's profile shows Online/Offline.

## Phase 3 — Reminders & scheduling (first "useful" milestone)
- Hermes ships a unified `cronjob` tool — no separate scheduler to build.
- Define the **reminders-scheduling** skill (see
  [`skills/reminders-scheduling/SKILL.md`](skills/reminders-scheduling/SKILL.md)) so the agent
  has a consistent playbook for creating/pausing/editing reminders and delivering them to
  Telegram.
- Validate: ask the agent in Telegram, "Every morning at 8am remind me to check my calendar,"
  and confirm the job fires and delivers back to the same chat.

## Phase 4 — Message management (second "useful" milestone)
- Define the **message-management** skill (see
  [`skills/message-management/SKILL.md`](skills/message-management/SKILL.md)) covering
  summarizing/triaging incoming messages, flagging anything needing a reply, and daily digests.
- Validate with a few real conversations/group chats over a couple of days; refine the skill
  based on what the agent gets wrong.

## Phase 5 — Memory & personalization
- Fill in `SOUL.md` with the assistant's identity/tone.
- Let `USER.md` / `MEMORY.md` accumulate naturally; periodically review what the agent has
  learned about you and correct anything wrong.
- Consider an external memory provider (Honcho, Mem0, etc.) only if the built-in memory proves
  insufficient — start with the default.

## Phase 6 — Hardening & production hygiene
- Review [Security](https://hermes-agent.nousresearch.com/docs/user-guide/security):
  dangerous-command approval, user authorization on Telegram (restrict who can talk to the bot),
  container isolation for shell commands if the agent runs arbitrary code.
- Set up `hermes update` backup behavior (`updates.pre_update_backup`) so updates are safe.
- Decide on a terminal backend (`local` vs `docker`) based on how much you trust letting the
  agent run shell commands directly on your machine.

## Phase 7 — Iterate
- Add more messaging platforms (Discord, Slack, etc.) only after Telegram + the two core
  skills are solid.
- Let the agent's self-improvement loop create new skills from real usage; periodically prune
  stale ones via the Curator.
- Revisit this roadmap and check off/expand phases as they're completed — this is a living
  document, not a one-time spec.

## Out of scope for this repo
This repo does not run or contain the Hermes Agent source. All install/runtime steps happen on
the user's Linux machine. This repo only tracks planning docs and reference config/skill files
to copy into `~/.hermes/`.
