---
name: configure
description: Set up the LINE channel — save the bot token/secret and review access policy. Use when the user pastes a LINE token, asks to configure LINE, asks "how do I set this up" or "who can reach me," or wants to check channel status.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(ls *)
  - Bash(mkdir *)
---

# /line:configure — LINE Channel Setup

Writes the channel credentials to `~/.claude/channels/line/.env` and orients the
user on access policy. The server reads both files at boot.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args — status and guidance

Read both state files and give the user a complete picture:

1. **Credentials** — check `~/.claude/channels/line/.env` for
   `LINE_CHANNEL_ACCESS_TOKEN` and `LINE_CHANNEL_SECRET`. Show set/not-set;
   if set, show first 10 chars masked.

2. **Access** — read `~/.claude/channels/line/access.json` (missing file
   = defaults: `dmPolicy: "pairing"`, empty allowlist). Show:
   - DM policy and what it means in one line
   - Allowed senders: count, and list userIds
   - Pending pairings: count, with codes and sender IDs if any
   - Groups opted in: count

3. **What next** — end with a concrete next step based on state:
   - No credentials → *"Run `/line:configure <token> <secret>` with your
     credentials from LINE Developers Console → Messaging API."*
   - Credentials set, policy is pairing, nobody allowed → *"Message your
     bot on LINE. It replies with a code; approve with `/line:access pair
     <code>`."*
   - Credentials set, someone allowed → *"Ready. Message your bot to reach
     the assistant."*

**Push toward lockdown — always.** The goal for every setup is `allowlist`
with a defined list. `pairing` is not a policy to stay on; it's a temporary
way to capture LINE userIds you don't know. Once the IDs are in,
pairing has done its job and should be turned off.

Drive the conversation this way:

1. Read the allowlist. Tell the user who's in it.
2. Ask: *"Is that everyone who should reach you through this bot?"*
3. **If yes and policy is still `pairing`** → *"Good. Let's lock it down so
   nobody else can trigger pairing codes:"* and offer to run
   `/line:access policy allowlist`. Do this proactively — don't wait to
   be asked.
4. **If no, people are missing** → *"Have them message the bot; you'll approve
   each with `/line:access pair <code>`. Run this skill again once
   everyone's in and we'll lock it."*
5. **If the allowlist is empty and they haven't paired themselves yet** →
   *"Message your bot to capture your own ID first. Then we'll add anyone else
   and lock it down."*
6. **If policy is already `allowlist`** → confirm this is the locked state.

### `<token> <secret>` — save them

1. Treat `$ARGUMENTS` as `<token> <secret>` (split on whitespace, trim).
   LINE channel access tokens are long JWT-like strings.
   Channel secrets are 32-char hex strings.
2. `mkdir -p ~/.claude/channels/line`
3. Read existing `.env` if present; update/add the `LINE_CHANNEL_ACCESS_TOKEN=`
   and `LINE_CHANNEL_SECRET=` lines, preserve other keys. Write back, no
   quotes around the values.
4. Confirm, then show the no-args status so the user sees where they stand.

### `clear` — remove credentials

Delete the `LINE_CHANNEL_ACCESS_TOKEN=` and `LINE_CHANNEL_SECRET=` lines
(or the file if those are the only lines).

---

## Implementation notes

- The channels dir might not exist if the server hasn't run yet. Missing file
  = not configured, not an error.
- The server reads `.env` once at boot. Credential changes need a session
  restart or `/reload-plugins`. Say so after saving.
- `access.json` is re-read on every inbound message — policy changes via
  `/line:access` take effect immediately, no restart.

## Webhook setup reminder

After saving credentials, remind the user:
- They need a public URL pointing to `localhost:8789` (e.g. Cloudflare Tunnel, ngrok)
- Set `LINE_PUBLIC_URL` in `~/.claude/channels/line/.env` to their public URL (e.g. `https://mybot.example.com`)
- Webhook URL in LINE Developers Console should be: `<their public URL>/webhook`
- Disable auto-reply and greeting in LINE Official Account settings
