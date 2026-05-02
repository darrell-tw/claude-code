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

> 順序が重要です: **`.env` はプラグインをインストールする前に用意してください**。MCP サーバーは初回起動時に `~/.claude/channels/line/.env` を読み込みます。ファイルが不完全だとサーバーは終了し、後の `/reload-plugins` でも再試行されません — 復旧には Claude Code を完全に再起動する必要があります。

### 1. LINE Developers Console

1. [LINE Developers](https://developers.line.biz/) で Messaging API チャンネルを作成
2. **Channel Access Token**（長期）と **Channel Secret** を取得
3. LINE Official Account の設定で自動応答とあいさつメッセージを無効化

### 2. パブリック URL（webhook トンネル）

`localhost:8789` にフォワードするパブリック URL が必要です。例:

```bash
# オプション A: Cloudflare Tunnel
cloudflared tunnel create line-claude
cloudflared tunnel route dns line-claude mybot.example.com
cloudflared tunnel run line-claude

# オプション B: ngrok
ngrok http 8789
```

トンネルは起動したまま維持してください。

### 3. 認証情報を書き込む

エディタで `~/.claude/channels/line/.env` を直接作成:

```
LINE_CHANNEL_ACCESS_TOKEN=あなたのtoken
LINE_CHANNEL_SECRET=あなたのsecret
LINE_PUBLIC_URL=https://mybot.example.com
```

```bash
chmod 600 ~/.claude/channels/line/.env
```

> スラッシュコマンドの `/line:configure <token> <secret>` も使えますが、認証情報がシェル履歴と Claude Code セッションのトランスクリプトに残ります。ファイルを直接編集する方が安全です。

### 4. LINE Console で Webhook URL を設定

LINE Developers Console → 該当チャンネル → **Messaging API**:

- **Webhook URL**: `https://mybot.example.com/webhook`
- **Use webhook**: ON
- **Verify** をクリック → `Success` が返ること

### 5. プラグインをインストール

```
/plugin marketplace add darrell-tw/claude-code
/plugin install line@darrell-tw-plugins
/reload-plugins
```

ステップ 1〜3 で `.env` を先に整えていれば、`/reload-plugins` でサーバーがクリーンに起動します。順序を守れていない場合は Claude Code を完全に終了して再起動してください。

### 6. アカウントのペアリング

1. LINE で bot にメッセージを送信
2. Bot が 6 桁のペアリングコードを返信
3. Claude Code で: `/line:access pair <code>`

### 7. ロックダウン

全員のペアリングが完了したら:

```
/line:access policy allowlist
```

これで allowlist にいない人からのメッセージはサーバーが破棄するため、見知らぬ送信者がペアリングコードを引き出せなくなります。

## 返信フォーマット

デフォルトはプレーンテキストで返信します。Flex Message（構造化レイアウト）を有効にするには:

```
/line:access set replyFormat flex
```

プレーンテキストに戻す: `/line:access set replyFormat text`

## ツール

| ツール | 説明 |
| --- | --- |
| `reply` | LINE メッセージに返信（replyToken → push フォールバック） |
| `push` | プッシュメッセージを送信（replyToken 不要） |
| `show_loading` | 入力中インジケーターを表示 |
| `get_profile` | ユーザー情報を取得: 表示名、アイコン、ステータスメッセージ |

## アーキテクチャ

![LINE Channel Plugin アーキテクチャ](docs/architecture-dark.png)

**インバウンド（上段）**: ユーザーのメッセージが LINE Platform 経由で Cloudflare Tunnel に届き、ローカルの `localhost:8789` Bun webhook サーバーに着地。`gate()` で署名検証とアクセスポリシーをチェックし、MCP stdio で Claude Code に通知します。

**アウトバウンド（下段）**: Claude が MCP ツール（`reply` / `push`）を呼び、LINE Messaging API に POST。LINE Platform からユーザーへメッセージが届きます。

## アクセス制御

詳細は [ACCESS.md](ACCESS.md) を参照してください。

## ライセンス

Apache-2.0
