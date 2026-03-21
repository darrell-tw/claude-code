# LINE — Access & Delivery

LINE bots receive messages from anyone who adds them as a friend. The **default policy is pairing**: an unknown sender gets a 6-character code in reply and their message is dropped. You run `/line:access pair <code>` from your assistant session to approve them. Once approved, their messages pass through.

All state lives in `~/.claude/channels/line/access.json`. The `/line:access` skill commands edit this file; the server re-reads it on every inbound message, so changes take effect without a restart. Set `LINE_ACCESS_MODE=static` to pin config to what was on disk at boot (pairing is unavailable in static mode since it requires runtime writes).

## At a glance

| | |
| --- | --- |
| Default policy | `pairing` |
| Sender ID | LINE userId (e.g. `U1234567890abcdef1234567890abcdef`) |
| Group key | LINE groupId (starts with `C`) |
| Config file | `~/.claude/channels/line/access.json` |

## DM policies

`dmPolicy` controls how DMs from senders not on the allowlist are handled.

| Policy | Behavior |
| --- | --- |
| `pairing` (default) | Reply with a pairing code, drop the message. Approve with `/line:access pair <code>`. |
| `allowlist` | Drop silently. No reply. Use this once everyone who needs access is already on the list. |
| `disabled` | Drop everything, including allowlisted users and groups. |

```
/line:access policy allowlist
```

## User IDs

LINE identifies users by permanent IDs like `U1234567890abcdef1234567890abcdef`. These are unique per bot (provider) and won't change. The allowlist stores these IDs.

Pairing captures the ID automatically. There's no easy way to look up a userId manually — use the pairing flow.

```
/line:access allow U1234567890abcdef1234567890abcdef
/line:access remove U1234567890abcdef1234567890abcdef
```

## Groups

Groups are off by default. Opt each one in individually, keyed on the groupId (starts with `C`). The bot must be a member of the group to receive messages.

```
/line:access group add C1234567890abcdef1234567890abcdef
```

With the default `requireMention: true`, the bot responds only when @mentioned or when a message matches `mentionPatterns`. Pass `--no-mention` to process every message, or `--allow id1,id2` to restrict which members can trigger it.

```
/line:access group add C1234... --no-mention
/line:access group add C1234... --allow U1234...,U5678...
/line:access group rm C1234...
```

## Mention detection

In groups with `requireMention: true`, any of the following triggers the bot:

- A structured @mention (LINE's mention entity)
- A match against any regex in `mentionPatterns`

Example regex setup:

```
/line:access set mentionPatterns '["@claude", "\\bassistant\\b"]'
```

## Delivery

Configure outbound behavior with `/line:access set <key> <value>`.

**`replyToMode`** controls whether the reply tool uses replyToken (free) or push (costs quota). `first` (default) uses replyToken for the first response per inbound message.

**`textChunkLimit`** sets the split threshold. LINE accepts up to 5000 characters per message.

**`chunkMode`** chooses the split strategy: `length` cuts exactly at the limit; `newline` prefers paragraph boundaries.

## Skill reference

| Command | Effect |
| --- | --- |
| `/line:access` | Print current state: policy, allowlist, pending pairings, enabled groups. |
| `/line:access pair a4f91c` | Approve pairing code `a4f91c`. Adds the sender to `allowFrom` and sends a confirmation on LINE. |
| `/line:access deny a4f91c` | Discard a pending code. The sender is not notified. |
| `/line:access allow U1234...` | Add a userId directly. |
| `/line:access remove U1234...` | Remove from the allowlist. |
| `/line:access policy allowlist` | Set `dmPolicy`. Values: `pairing`, `allowlist`, `disabled`. |
| `/line:access group add C1234...` | Enable a group. Flags: `--no-mention`, `--allow id1,id2`. |
| `/line:access group rm C1234...` | Disable a group. |
| `/line:access set textChunkLimit 3000` | Set a config key: `replyToMode`, `textChunkLimit`, `chunkMode`, `mentionPatterns`. |

## Config file

`~/.claude/channels/line/access.json`. Absent file is equivalent to `pairing` policy with empty lists, so the first DM triggers pairing.

```jsonc
{
  // Handling for DMs from senders not in allowFrom.
  "dmPolicy": "pairing",

  // LINE userIds allowed to DM.
  "allowFrom": ["U1234567890abcdef1234567890abcdef"],

  // Groups the bot is active in. Empty object = DM-only.
  "groups": {
    "C1234567890abcdef1234567890abcdef": {
      // true: respond only to @mentions and pattern matches.
      "requireMention": true,
      // Restrict triggers to these senders. Empty = any member.
      "allowFrom": []
    }
  },

  // Case-insensitive regexes that count as a mention.
  "mentionPatterns": ["@claude"],

  // Threading on chunked replies: first | all | off
  "replyToMode": "first",

  // Split threshold. LINE accepts up to 5000.
  "textChunkLimit": 5000,

  // length = cut at limit. newline = prefer paragraph boundaries.
  "chunkMode": "newline"
}
```
