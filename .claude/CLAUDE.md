# LINE Channel Plugin — Development Guide

## Architecture

```
LINE App
  ↓ (webhook POST)
LINE Platform
  ↓
Public URL (Cloudflare Tunnel / ngrok / etc) → localhost:8789
  ↓
Bun HTTP Server (server.ts, port 8789)
  ↓ gate() validates sender
MCP notification (stdio)
  ↓
Claude Code Session ← must be started by Claude Code to have stdio connection
  ↓ (tool call)
LINE Messaging API → LINE App
```

**server.ts is both an HTTP server (receives webhooks) and an MCP server (connects to Claude Code).**
If not started by Claude Code, HTTP works but MCP is disconnected.

## Channel Principles

1. **stdout is sacred** — never use `console.log()`, use `process.stderr.write()`
2. **Validate sender not room** — `gate()` validates `source.userId`, not `chatId`
3. **State must be persisted** — access.json uses atomic write (tmp + rename)

## Troubleshooting

### Messages not received
- Confirm started via Claude Code plugin system (not standalone `bun server.ts`)
- Confirm port 8789 is not occupied by orphan process
- Confirm public URL tunnel is running

### Port 8789 occupied
Previous session exit may leave orphan bun process. `lsof -i :8789` to find PID, `kill` it.

### replyToken expired
LINE replyToken is valid for ~30 seconds. If Claude takes too long, server.ts auto-falls back to push API (costs quota).

### Loading animation not supported in groups
LINE's `/v2/bot/chat/loading/start` only works in 1-on-1 chats. server.ts handles this (only calls when `source.type === 'user'`).

### Group @mention detection
LINE's `mention.mentionees[].isSelf` is the field to detect bot being @mentioned. Do not use `type === 'user'` (matches any user mention).

## File Structure

```
line/
├── .claude-plugin/
│   └── plugin.json        # Plugin manifest
├── .mcp.json              # MCP server config (uses CLAUDE_PLUGIN_ROOT)
├── server.ts              # Core: MCP + HTTP server
├── package.json           # Only depends on @modelcontextprotocol/sdk
├── skills/
│   ├── access/            # /line:access skill
│   └── configure/         # /line:configure skill
├── ACCESS.md              # Access control docs
└── README.md              # English README
```

## Runtime Paths

| Purpose | Path |
|---------|------|
| Credentials | `~/.claude/channels/line/.env` |
| Access control | `~/.claude/channels/line/access.json` |
| Attachments | `~/.claude/channels/line/inbox/` |
| Pairing approvals | `~/.claude/channels/line/approved/` |

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `LINE_CHANNEL_ACCESS_TOKEN` | Yes | From LINE Developers Console |
| `LINE_CHANNEL_SECRET` | Yes | From LINE Developers Console |
| `LINE_PUBLIC_URL` | Recommended | Public URL for file sharing (e.g. `https://mybot.example.com`) |
| `LINE_WEBHOOK_PORT` | No | Default: 8789 |
| `LINE_ACCESS_MODE` | No | Set to `static` to skip signature verification |
