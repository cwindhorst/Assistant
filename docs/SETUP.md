# Setup Guide

Follow these steps **on the Linux machine** where Hermes Agent is installed. This repo only
holds reference files — copy the relevant pieces into `~/.hermes/`.

## 1. Confirm/refresh the install

```bash
hermes update
hermes config check
```

If Hermes isn't installed yet:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

## 2. Configure the xAI (Grok) provider

1. Grab an API key from the xAI console.
2. Add it as a secret (never commit this):
   ```bash
   hermes config set XAI_API_KEY sk-xai-...
   ```
   This writes to `~/.hermes/.env`.
3. Pick the model:
   ```bash
   hermes model
   ```
   Select the xAI/Grok option, or set it directly once you know the model id:
   ```bash
   hermes config set model xai/<grok-model-id>
   ```
4. Sanity-check before wiring any platform:
   ```bash
   hermes chat
   ```

Reference: [`config/config.yaml.example`](../config/config.yaml.example),
[`config/.env.example`](../config/.env.example).

## 3. Create the Telegram bot

1. In Telegram, message **@BotFather** → `/newbot`.
2. Choose a display name and a username ending in `bot`.
3. Copy the API token BotFather gives you.
4. Run the interactive wizard — this is the easiest path, it prompts for the token and
   writes it to `~/.hermes/.env` for you:
   ```bash
   hermes gateway setup
   ```
   Prefer to wire it by hand instead? Store the token as a secret and merge the
   `platforms.telegram` block from
   [`config/config.yaml.example`](../config/config.yaml.example) into
   `~/.hermes/config.yaml`:
   ```bash
   hermes config set TELEGRAM_BOT_TOKEN 123456789:ABC...
   ```
5. (Optional, group use only) Message @BotFather → `/mybots` → your bot → **Bot Settings →
   Group Privacy → Turn off**, then remove/re-add the bot to any group so it can see all
   messages, not just `/commands`.

## 4. Start the gateway and verify

```bash
hermes gateway
```

Message your bot from Telegram. Confirm:
- The bot replies using the xAI/Grok model.
- Voice memos get transcribed (if you plan to use voice).
- `hermes gateway status` (or platform logs in `~/.hermes/logs/gateway.log`) shows a healthy
  Telegram connection.

## 5. Install the starter skills

Copy the skill folders from this repo's `skills/` directory into `~/.hermes/skills/`:

```bash
cp -r skills/reminders-scheduling ~/.hermes/skills/
cp -r skills/message-management ~/.hermes/skills/
cp -r skills/web-research ~/.hermes/skills/
```

Then in a chat with the agent, ask it to review and start using them — Hermes' skill system
is designed for the agent to read `SKILL.md` on demand, not for you to hand-run anything.

## 6. Validate the first two use cases

- **Reminders**: "Every morning at 8am, remind me to check my calendar." Confirm the job is
  created (`hermes cron list`, or `/cron list` in chat) and fires on schedule, delivering to
  the same Telegram chat.
- **Message management**: Have a few real conversations/group chats over a day or two, then
  ask for a daily digest or "what did I miss today?" and check the summary quality.

## 7. Harden before relying on it daily

Read [Security](https://hermes-agent.nousresearch.com/docs/user-guide/security) and:
- Restrict who can message the bot (Telegram user allowlist).
- Decide the terminal backend (`local` vs `docker`) based on how much shell access you want
  the agent to have.
- Turn on `updates.pre_update_backup` so `hermes update` is safe to run unattended.
