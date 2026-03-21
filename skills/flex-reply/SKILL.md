---
name: flex-reply
description: Reply to LINE messages using Flex Message format instead of plain text. Provides a structured bubble template with header, body, and footer.
user-invocable: false
---

# Flex Reply — Structured LINE Responses

When replying to LINE messages, prefer `send_flex` over plain text `reply` for a richer experience.

## Template

Use this bubble structure as the base for all replies:

```json
{
  "type": "bubble",
  "header": {
    "type": "box",
    "layout": "vertical",
    "contents": [
      { "type": "text", "text": "TITLE", "weight": "bold", "size": "lg", "color": "#1a1a1a" }
    ],
    "paddingBottom": "sm"
  },
  "body": {
    "type": "box",
    "layout": "vertical",
    "contents": [
      { "type": "text", "text": "Main content goes here.", "wrap": true, "size": "sm", "color": "#333333" }
    ],
    "spacing": "md"
  },
  "footer": {
    "type": "box",
    "layout": "horizontal",
    "contents": [
      { "type": "text", "text": "📨 0/200", "size": "xxs", "color": "#999999", "align": "end" }
    ],
    "paddingTop": "sm"
  }
}
```

## Usage Guidelines

- **Header**: Short title summarizing the response (omit if unnecessary)
- **Body**: Main content — use multiple text elements, boxes, or separators to structure information
- **Footer**: Status line — show quota usage via `get_quota` tool as `📨 used/total`
- **alt_text**: Always set a meaningful preview (shown in notifications)
- **Wrap**: Always set `"wrap": true` on text that may be long
- For lists or key-value data, use horizontal boxes:
  ```json
  {
    "type": "box", "layout": "horizontal",
    "contents": [
      { "type": "text", "text": "Label", "size": "sm", "color": "#999999", "flex": 2 },
      { "type": "text", "text": "Value", "size": "sm", "color": "#333333", "flex": 5, "wrap": true }
    ]
  }
  ```
- For visual separation, use `{ "type": "separator", "margin": "md" }`

## When to Use Plain Text Instead

- Very short confirmations (e.g. "OK", "Done")
- Error messages that need no formatting
- When the user explicitly asks for plain text
