# LINE Channel for Claude Code

[English](README.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md)

ปลั๊กอิน LINE Messaging API แบบครบฟังก์ชันสำหรับ Claude Code — สะพานรับส่งข้อความพร้อมระบบควบคุมการเข้าถึงในตัว

## คุณสมบัติ

- **รับส่งข้อความสองทาง**: ส่งและรับข้อความผ่าน LINE
- **ควบคุมการเข้าถึง**: ระบบจับคู่ (pairing), รายการอนุญาต (allowlist), รองรับกลุ่มด้วยการ @mention
- **ตอบกลับอัจฉริยะ**: ใช้ replyToken ฟรีเมื่อมี, สลับเป็น push อัตโนมัติเมื่อหมดอายุ
- **แบ่งข้อความอัตโนมัติ**: ข้อความยาวแบ่งที่ 5000 ตัวอักษร โดยตัดที่ขอบย่อหน้า
- **แอนิเมชัน Loading**: แสดงสถานะกำลังพิมพ์ขณะ Claude ประมวลผล
- **รองรับไฟล์แนบ**: รูปภาพ, วิดีโอ, เสียง, ไฟล์ดาวน์โหลดอัตโนมัติไปยัง inbox
- **ความปลอดภัย Webhook**: ตรวจสอบลายเซ็น HMAC-SHA256 ทุกเหตุการณ์ขาเข้า

## สิ่งที่ต้องมี

- [Bun](https://bun.sh) runtime
- URL สาธารณะชี้ไปที่ localhost:8789 (เช่น [cloudflared](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/), [ngrok](https://ngrok.com/))
- บัญชี LINE Developers ที่สร้าง Messaging API channel แล้ว

> **สำคัญ**: อย่ารัน `bun server.ts` โดยตรง! LINE Channel เป็น MCP Server ต้องเริ่มผ่านระบบ plugin ของ Claude Code เท่านั้นจึงจะรับข้อความได้

## การตั้งค่า

### 1. LINE Developers Console

1. สร้าง Messaging API channel ที่ [LINE Developers](https://developers.line.biz/)
2. รับ **Channel Access Token** (แบบถาวร) และ **Channel Secret**
3. ปิดข้อความตอบกลับอัตโนมัติและข้อความต้อนรับใน LINE Official Account

### 2. ตั้งค่า credentials

```
/line:configure <token> <secret>
```

หรือสร้างไฟล์ `~/.claude/channels/line/.env` เอง:

```
LINE_CHANNEL_ACCESS_TOKEN=token-ของคุณ
LINE_CHANNEL_SECRET=secret-ของคุณ
```

### 3. URL สาธารณะ (webhook tunnel)

คุณต้องมี URL สาธารณะที่ forward ไปยัง `localhost:8789` เช่น:

```bash
# ตัวเลือก A: Cloudflare Tunnel
cloudflared tunnel create line-claude
cloudflared tunnel route dns line-claude mybot.example.com
cloudflared tunnel run line-claude

# ตัวเลือก B: ngrok
ngrok http 8789
```

จากนั้น:
1. เพิ่ม `LINE_PUBLIC_URL=https://mybot.example.com` ใน `~/.claude/channels/line/.env`
2. ตั้ง webhook URL ใน LINE Developers Console: `https://mybot.example.com/webhook`

### 4. จับคู่บัญชี

1. ติดตั้ง plugin แล้วเริ่ม Claude Code session
2. ส่งข้อความหา bot ของคุณทาง LINE
3. Bot จะตอบกลับด้วยรหัสจับคู่
4. ใน Claude Code: `/line:access pair <code>`

### 5. ล็อกการเข้าถึง

เมื่อจับคู่ทุกคนเรียบร้อยแล้ว:

```
/line:access policy allowlist
```

## เครื่องมือ

| เครื่องมือ | รายละเอียด |
| --- | --- |
| `reply` | ตอบกลับข้อความ LINE (replyToken → สลับเป็น push) |
| `push` | ส่งข้อความแบบ push (ไม่ต้องใช้ replyToken) |
| `show_loading` | แสดงสถานะกำลังพิมพ์ |
| `get_profile` | ดึงข้อมูลผู้ใช้: ชื่อ, รูปโปรไฟล์, ข้อความสถานะ |

## สถาปัตยกรรม

```
LINE App → LINE Platform → Tunnel → Bun HTTP (localhost:8789) → gate() → MCP notification → Claude Code
Claude Code → MCP tool call → LINE Messaging API → LINE App
```

## การควบคุมการเข้าถึง

ดูเอกสารฉบับเต็มที่ [ACCESS.md](ACCESS.md)

## สัญญาอนุญาต

Apache-2.0
