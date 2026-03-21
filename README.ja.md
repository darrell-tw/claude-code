# LINE Channel for Claude Code

[English](README.md) | [繁體中文](README.zh-TW.md) | [ภาษาไทย](README.th.md)

Claude Code 用フル機能 LINE Messaging API チャンネルプラグイン — アクセス制御機能付きメッセージングブリッジ。

## 機能

- **双方向メッセージング**: LINE 経由でテキストメッセージの送受信
- **アクセス制御**: ペアリングフロー、許可リスト、@メンション対応のグループサポート
- **スマートリプライ**: 無料の replyToken を優先使用、期限切れ時は push に自動フォールバック
- **自動分割**: 長文メッセージを5000文字で段落境界を考慮して分割
- **ローディングアニメーション**: Claude 処理中に入力中インジケーターを表示
- **添付ファイル対応**: 写真、動画、音声、ファイルを inbox に自動ダウンロード
- **Webhook セキュリティ**: 全受信イベントに HMAC-SHA256 署名検証

## 前提条件

- [Bun](https://bun.sh) ランタイム
- localhost:8789 を指すパブリック URL（例: [cloudflared](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)、[ngrok](https://ngrok.com/)）
- Messaging API チャンネルを作成済みの LINE Developers アカウント

> **重要**: `bun server.ts` を直接実行しないでください！LINE Channel は MCP Server であり、メッセージを受信するには Claude Code のプラグインシステムから起動する必要があります。

## セットアップ

### 1. LINE Developers Console

1. [LINE Developers](https://developers.line.biz/) で Messaging API チャンネルを作成
2. **Channel Access Token**（長期）と **Channel Secret** を取得
3. LINE Official Account の設定で自動応答とあいさつメッセージを無効化

### 2. 認証情報の設定

```
/line:configure <token> <secret>
```

または `~/.claude/channels/line/.env` を手動作成:

```
LINE_CHANNEL_ACCESS_TOKEN=あなたのtoken
LINE_CHANNEL_SECRET=あなたのsecret
```

### 3. パブリック URL（webhook トンネル）

`localhost:8789` にフォワードするパブリック URL が必要です。例:

```bash
# オプション A: Cloudflare Tunnel
cloudflared tunnel create line-claude
cloudflared tunnel route dns line-claude mybot.example.com
cloudflared tunnel run line-claude

# オプション B: ngrok
ngrok http 8789
```

その後:
1. `~/.claude/channels/line/.env` に `LINE_PUBLIC_URL=https://mybot.example.com` を追加
2. LINE Developers Console で Webhook URL を設定: `https://mybot.example.com/webhook`

### 4. アカウントのペアリング

1. プラグインをインストールして Claude Code セッションを開始
2. LINE で bot にメッセージを送信
3. Bot がペアリングコードを返信
4. Claude Code で: `/line:access pair <code>`

### 5. ロックダウン

全員のペアリングが完了したら:

```
/line:access policy allowlist
```

## ツール

| ツール | 説明 |
| --- | --- |
| `reply` | LINE メッセージに返信（replyToken → push フォールバック） |
| `push` | プッシュメッセージを送信（replyToken 不要） |
| `show_loading` | 入力中インジケーターを表示 |
| `get_profile` | ユーザー情報を取得: 表示名、アイコン、ステータスメッセージ |

## アーキテクチャ

```
LINE App → LINE Platform → Tunnel → Bun HTTP (localhost:8789) → gate() → MCP notification → Claude Code
Claude Code → MCP tool call → LINE Messaging API → LINE App
```

## アクセス制御

詳細は [ACCESS.md](ACCESS.md) を参照してください。

## ライセンス

Apache-2.0
