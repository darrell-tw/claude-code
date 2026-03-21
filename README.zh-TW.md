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

### 1. LINE Developers Console

1. 在 [LINE Developers](https://developers.line.biz/) 建立 Messaging API channel
2. 取得 **Channel Access Token**（長效）和 **Channel Secret**
3. 在 LINE Official Account 設定中關閉自動回覆和歡迎訊息

### 2. 設定憑證

```
/line:configure <token> <secret>
```

或手動建立 `~/.claude/channels/line/.env`：

```
LINE_CHANNEL_ACCESS_TOKEN=你的-token
LINE_CHANNEL_SECRET=你的-secret
```

### 3. 公開 URL（webhook tunnel）

你需要一個指向 `localhost:8789` 的公開 URL，例如：

```bash
# 方案 A：Cloudflare Tunnel
cloudflared tunnel create line-claude
cloudflared tunnel route dns line-claude mybot.example.com
cloudflared tunnel run line-claude

# 方案 B：ngrok
ngrok http 8789
```

接著：
1. 在 `~/.claude/channels/line/.env` 加入 `LINE_PUBLIC_URL=https://mybot.example.com`
2. 在 LINE Developers Console 設定 Webhook URL：`https://mybot.example.com/webhook`

### 4. 配對帳號

1. 安裝 plugin 後啟動 Claude Code session
2. 用 LINE 傳訊息給你的 bot
3. Bot 回覆配對碼
4. 在 Claude Code 中：`/line:access pair <code>`

### 5. 鎖定

所有人都配對完成後：

```
/line:access policy allowlist
```

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
