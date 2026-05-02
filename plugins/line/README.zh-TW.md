# LINE Channel for Claude Code

[English](README.md) | [ภาษาไทย](README.th.md) | [日本語](README.ja.md)

全功能 LINE Messaging API 頻道外掛，用於 Claude Code — 內建存取控制的訊息橋接器。

## 功能

- **雙向訊息**：透過 LINE 收發文字訊息
- **存取控制**：配對流程、白名單、群組支援（@mention 觸發）
- **智慧回覆**：優先使用免費的 replyToken，過期自動改用 push
- **自動分段**：長訊息在 5000 字處分段，優先在段落邊界切割
- **Loading 動畫**：Claude 處理中顯示打字指示器
- **附件支援**：照片、影片、音訊、檔案自動下載到 inbox
- **Webhook 安全**：所有入站事件做 HMAC-SHA256 簽名驗證

## 前置條件

- [Bun](https://bun.sh) runtime
- 公開 URL 指向 localhost:8789（例如 [cloudflared](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)、[ngrok](https://ngrok.com/)）
- LINE Developers 帳號，已建立 Messaging API channel

> **重要**：不要直接 `bun server.ts`！LINE Channel 是 MCP Server，必須透過 Claude Code plugin 系統啟動才能接收訊息。

## 首次設定

> 順序很重要：**`.env` 要在安裝 plugin 之前先寫好**。MCP server 第一次啟動時會讀 `~/.claude/channels/line/.env`，如果檔案缺漏，server 會直接 exit，後續 `/reload-plugins` 也不會重試 — 必須完整重啟 Claude Code 才能恢復。

### 1. LINE Developers Console

1. 在 [LINE Developers](https://developers.line.biz/) 建立 Messaging API channel
2. 取得 **Channel Access Token**（長效）和 **Channel Secret**
3. 在 LINE Official Account 設定中關閉自動回覆和歡迎訊息

### 2. 公開 URL（webhook tunnel）

你需要一個指向 `localhost:8789` 的公開 URL，例如：

```bash
# 方案 A：Cloudflare Tunnel
cloudflared tunnel create line-claude
cloudflared tunnel route dns line-claude mybot.example.com
cloudflared tunnel run line-claude

# 方案 B：ngrok
ngrok http 8789
```

讓 tunnel 持續跑著。

### 3. 寫入憑證

直接用編輯器建立 `~/.claude/channels/line/.env`：

```
LINE_CHANNEL_ACCESS_TOKEN=你的-token
LINE_CHANNEL_SECRET=你的-secret
LINE_PUBLIC_URL=https://mybot.example.com
```

```bash
chmod 600 ~/.claude/channels/line/.env
```

> 也有 slash command 寫法 `/line:configure <token> <secret>`，但 token 跟 secret 會留在 shell history 跟 Claude Code session transcript 裡。直接編檔比較乾淨。

### 4. 在 LINE Console 設 Webhook URL

LINE Developers Console → 你的 channel → **Messaging API**：

- **Webhook URL**：`https://mybot.example.com/webhook`
- **Use webhook**：開啟
- 點 **Verify** 應該回 `Success`

### 5. 安裝 plugin

```
/plugin marketplace add darrell-tw/claude-code
/plugin install line@darrell-tw-plugins
/reload-plugins
```

如果你照著步驟 1–3 先寫好 `.env`，這裡 `/reload-plugins` 就會把 server 啟起來。如果順序不對，要完整退出再重開 Claude Code。

### 6. 配對帳號

1. 用 LINE 傳訊息給你的 bot
2. Bot 回覆 6 碼配對碼
3. 在 Claude Code 中：`/line:access pair <code>`

### 7. 鎖定

所有人都配對完成後：

```
/line:access policy allowlist
```

之後陌生人傳訊息給 bot 會直接被 server drop，不再產生配對碼。

## 回覆格式

預設使用純文字回覆。啟用 Flex Message 結構化回覆：

```
/line:access set replyFormat flex
```

切回純文字：`/line:access set replyFormat text`

## 工具

| 工具 | 說明 |
| --- | --- |
| `reply` | 回覆 LINE 訊息（replyToken → push 備援） |
| `push` | 推送訊息（不需要 replyToken） |
| `show_loading` | 顯示打字指示器 |
| `get_profile` | 取得用戶資料：暱稱、頭像、狀態訊息 |

## 架構

```
LINE App → LINE Platform → Cloudflare Tunnel → Bun HTTP (localhost:8789) → gate() → MCP notification → Claude Code
Claude Code → MCP tool call → LINE Messaging API → LINE App
```

## 存取控制

完整說明請參考 [ACCESS.md](ACCESS.md)。

## 授權

Apache-2.0
