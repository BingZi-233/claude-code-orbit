# OneBot V11 — Access & Delivery

A QQ bot is addressable by anyone who knows the QQ number. Without a gate, any message would flow into your assistant session. The access model described here decides who gets through.

By default, a private message from an unknown sender triggers **pairing**: the bot replies with a 6-character code and drops the message. You run `/onebot:access pair <code>` from your assistant session to approve them. Once approved, their messages pass through.

All state lives in `~/.claude/channels/onebot/access.json`. The `/onebot:access` skill commands edit this file; the server re-reads it on every inbound message, so changes take effect without a restart.

## At a glance

| | |
| --- | --- |
| Default policy | `pairing` |
| Sender ID | QQ number (e.g. `123456789`) |
| Group key | QQ group number (e.g. `987654321`) |
| Config file | `~/.claude/channels/onebot/access.json` |

## DM policies

`dmPolicy` controls how private messages from senders not on the allowlist are handled.

| Policy | Behavior |
| --- | --- |
| `pairing` (default) | Reply with a pairing code, drop the message. Approve with `/onebot:access pair <code>`. |
| `allowlist` | Drop silently. No reply. |
| `disabled` | Drop everything, including allowlisted users and groups. |

```
/onebot:access policy allowlist
```

## User IDs

QQ identifies users by **numeric QQ numbers** like `123456789`. The allowlist stores QQ number strings.

Pairing captures the QQ number automatically.

```
/onebot:access allow 123456789
/onebot:access remove 123456789
```

## Groups

Groups are off by default. Opt each one in individually.

```
/onebot:access group add 987654321
```

With the default `requireMention: true`, the bot responds only when @mentioned. Pass `--no-mention` to process every message, or `--allow id1,id2` to restrict which members can trigger it.

```
/onebot:access group add 987654321 --no-mention
/onebot:access group add 987654321 --allow 123456789,234567890
/onebot:access group rm 987654321
```

## Mention detection

In groups with `requireMention: true`, any of the following triggers the bot:

- An `[CQ:at,qq=botId]` segment in the message
- A reply to one of the bot's messages
- A match against any regex in `mentionPatterns`

```
/onebot:access set mentionPatterns '["claude", "assistant"]'
```

## Delivery

Configure outbound behavior with `/onebot:access set <key> <value>`.

**`textChunkLimit`** sets the split threshold. Default: 2000 characters.

**`chunkMode`** chooses the split strategy: `length` cuts exactly at the limit; `newline` prefers paragraph boundaries.

## Skill reference

| Command | Effect |
| --- | --- |
| `/onebot:access` | Print current state: policy, allowlist, pending pairings, enabled groups. |
| `/onebot:access pair a4f91c` | Approve pairing code `a4f91c`. |
| `/onebot:access deny a4f91c` | Discard a pending code. |
| `/onebot:access allow 123456789` | Add a QQ number directly. |
| `/onebot:access remove 123456789` | Remove from the allowlist. |
| `/onebot:access policy allowlist` | Set `dmPolicy`. Values: `pairing`, `allowlist`, `disabled`. |
| `/onebot:access group add 987654321` | Enable a group. Flags: `--no-mention`, `--allow id1,id2`. |
| `/onebot:access group rm 987654321` | Disable a group. |
| `/onebot:access set textChunkLimit 3000` | Set a config key. |

## Config file

`~/.claude/channels/onebot/access.json`:

```jsonc
{
  // Handling for DMs from senders not in allowFrom.
  "dmPolicy": "pairing",

  // QQ numbers allowed to DM.
  "allowFrom": ["123456789"],

  // Groups the bot is active in. Empty object = DM-only.
  "groups": {
    "987654321": {
      "requireMention": true,
      "allowFrom": []
    }
  },

  // Case-insensitive regexes that count as a mention.
  "mentionPatterns": ["claude"],

  // Split threshold.
  "textChunkLimit": 2000,

  // length = cut at limit. newline = prefer paragraph boundaries.
  "chunkMode": "newline"
}
```
