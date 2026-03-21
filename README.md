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

### 1. LINE Developers Console

1. Create a Messaging API channel at [LINE Developers](https://developers.line.biz/)
2. Get your **Channel Access Token** (long-lived) and **Channel Secret**
3. Disable auto-reply and greeting messages in LINE Official Account settings

### 2. Configure credentials

```
/line:configure <token> <secret>
```

Or manually create `~/.claude/channels/line/.env`:

```
LINE_CHANNEL_ACCESS_TOKEN=your-token-here
LINE_CHANNEL_SECRET=your-secret-here
```

### 3. Public URL (webhook tunnel)

You need a public URL that forwards to `localhost:8789`. Examples:

```bash
# Option A: Cloudflare Tunnel
cloudflared tunnel create line-claude
cloudflared tunnel route dns line-claude mybot.example.com
cloudflared tunnel run line-claude

# Option B: ngrok
ngrok http 8789
```

Then:
1. Add `LINE_PUBLIC_URL=https://mybot.example.com` to `~/.claude/channels/line/.env`
2. Set webhook URL in LINE Developers Console: `https://mybot.example.com/webhook`

### 4. Pair your account

1. Message your bot on LINE
2. Bot replies with a pairing code
3. In Claude Code: `/line:access pair <code>`

### 5. Lock down

Once everyone is paired:

```
/line:access policy allowlist
```

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
