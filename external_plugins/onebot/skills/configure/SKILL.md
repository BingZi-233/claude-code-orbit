---
name: configure
description: Set up the OneBot V11 channel — save the WebSocket URL and review access policy. Use when the user asks to configure OneBot/QQ, asks "how do I set this up", or wants to check channel status.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(ls *)
  - Bash(mkdir *)
---

# /onebot:configure — OneBot V11 Channel Setup

Writes the WebSocket URL to `~/.claude/channels/onebot/.env` and orients the
user on access policy. The server reads the .env file at boot.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args — status and guidance

Read both state files and give the user a complete picture:

1. **WebSocket URL** — check `~/.claude/channels/onebot/.env` for
   `ONEBOT_WS_URL`. Show set/not-set; if set, show the URL.
   Also check `ONEBOT_ACCESS_TOKEN` (show set/not-set, masked).

2. **Access** — read `~/.claude/channels/onebot/access.json` (missing file
   = defaults: `dmPolicy: "pairing"`, empty allowlist). Show:
   - DM policy and what it means in one line
   - Allowed senders: count, and list QQ numbers
   - Pending pairings: count, with codes and sender IDs if any

3. **What next** — end with a concrete next step based on state:
   - No URL → *"Run `/onebot:configure ws://127.0.0.1:3001` with your
     LLOneBot WebSocket address."*
   - URL set, policy is pairing, nobody allowed → *"Send a private message
     to your QQ bot. It replies with a code; approve with `/onebot:access pair
     <code>`."*
   - URL set, someone allowed → *"Ready. Message your bot to reach the
     assistant."*

**Push toward lockdown — always.** Once IDs are captured via pairing,
switch to `allowlist` mode.

### `<url>` — save WebSocket URL

1. Treat first argument as the WebSocket URL (trim whitespace).
   Typical format: `ws://127.0.0.1:3001`
2. `mkdir -p ~/.claude/channels/onebot`
3. Read existing `.env` if present; update/add the `ONEBOT_WS_URL=` line,
   preserve other keys. Write back.
4. Confirm, then show the no-args status.

### `<url> <token>` — save URL and access token

1. Save `ONEBOT_WS_URL=<url>` and `ONEBOT_ACCESS_TOKEN=<token>` to `.env`.

### `clear` — remove config

Delete the `ONEBOT_WS_URL=` and `ONEBOT_ACCESS_TOKEN=` lines
(or the file if those are the only lines).

---

## Implementation notes

- The channels dir might not exist if the server hasn't run yet. Missing file
  = not configured, not an error.
- The server reads `.env` once at boot. Config changes need a session restart
  or `/reload-plugins`. Say so after saving.
- `access.json` is re-read on every inbound message — policy changes via
  `/onebot:access` take effect immediately, no restart.
