# LINE Channel for Claude Code

[繁體中文](README.zh-TW.md) | [ภาษาไทย](README.th.md) | [日本語](README.ja.md)

A full-featured LINE Messaging API channel plugin for Claude Code — messaging bridge with built-in access control.

## Features

- **Bidirectional messaging**: Send and receive text messages via LINE
- **Access control**: Pairing flow, allowlists, group support with mention-triggering
- **Smart reply**: Uses free replyToken when available, falls back to push
- **Auto-chunking**: Long messages split at 5000 chars with paragraph-aware breaks
- **Loading animation**: Shows typing indicator while Claude processes
- **Attachment support**: Photos, videos, audio, files auto-downloaded to inbox
- **Webhook security**: HMAC-SHA256 signature verification on all inbound events

## Prerequisites

- [Bun](https://bun.sh) runtime
- A public URL pointing to localhost:8789 (e.g. [cloudflared](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/), [ngrok](https://ngrok.com/))
- LINE Developers account with a Messaging API channel

## Setup

> Set up `.env` **before** installing the plugin. The MCP server reads `~/.claude/channels/line/.env` on first spawn; if it's missing or incomplete, the server exits and won't auto-retry on later `/reload-plugins` — you'd have to fully restart Claude Code to recover.

### 1. LINE Developers Console

1. Create a Messaging API channel at [LINE Developers](https://developers.line.biz/)
2. Get your **Channel Access Token** (long-lived) and **Channel Secret**
3. Disable auto-reply and greeting messages in LINE Official Account settings

### 2. Public URL (webhook tunnel)

You need a public URL that forwards to `localhost:8789`. Examples:

```bash
# Option A: Cloudflare Tunnel
cloudflared tunnel create line-claude
cloudflared tunnel route dns line-claude mybot.example.com
cloudflared tunnel run line-claude

# Option B: ngrok
ngrok http 8789
```

Keep the tunnel running.

### 3. Write credentials

Create `~/.claude/channels/line/.env` directly with your editor:

```
LINE_CHANNEL_ACCESS_TOKEN=your-token-here
LINE_CHANNEL_SECRET=your-secret-here
LINE_PUBLIC_URL=https://mybot.example.com
```

```bash
chmod 600 ~/.claude/channels/line/.env
```

> A slash command alternative `/line:configure <token> <secret>` exists but puts the credentials into your shell history and Claude Code session transcript. Editing the file directly keeps them out.

### 4. Set webhook URL in LINE Console

LINE Developers Console → your channel → **Messaging API**:

- **Webhook URL**: `https://mybot.example.com/webhook`
- **Use webhook**: ON
- Click **Verify** — should return `Success`

### 5. Install the plugin

```
/plugin marketplace add darrell-tw/claude-code
/plugin install line@darrell-tw-plugins
/reload-plugins
```

If you set up `.env` first (steps 1–3), the MCP server will spawn cleanly on `/reload-plugins`. Otherwise, fully quit and reopen Claude Code.

### 6. Pair your account

1. Message your bot on LINE
2. Bot replies with a pairing code
3. In Claude Code: `/line:access pair <code>`

### 7. Lock down

Once everyone is paired:

```
/line:access policy allowlist
```

This drops messages from anyone not in the allowlist, so unknown senders no longer trigger pairing codes.

## Reply Format

By default, the bot replies with plain text. To enable rich Flex Message replies:

```
/line:access set replyFormat flex
```

To switch back: `/line:access set replyFormat text`

## Tools

| Tool | Description |
| --- | --- |
| `reply` | Reply to a LINE message (replyToken → push fallback) |
| `push` | Send a push message (no replyToken needed) |
| `show_loading` | Show typing indicator |
| `get_profile` | Get user's display name, picture, status |

## Architecture

```
LINE App → LINE Platform → Cloudflare Tunnel → Bun HTTP (localhost:8789) → gate() → MCP notification → Claude Code
Claude Code → MCP tool call → LINE Messaging API → LINE App
```

## Access Control

See [ACCESS.md](ACCESS.md) for full documentation.

## License

Apache-2.0
