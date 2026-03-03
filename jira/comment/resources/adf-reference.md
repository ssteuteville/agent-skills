# Atlassian Document Format (ADF) Quick Reference

ADF is the JSON format Jira uses for rich text. Use it instead of Markdown to produce properly rendered comments in the Jira UI.

## Document Skeleton

Every ADF document starts with:

```json
{
  "version": 1,
  "type": "doc",
  "content": [ ...top-level block nodes... ]
}
```

## Block Nodes

### Paragraph

```json
{ "type": "paragraph", "content": [ ...inline nodes... ] }
```

### Heading (levels 1–6)

```json
{ "type": "heading", "attrs": { "level": 2 }, "content": [ ...inline nodes... ] }
```

### Bullet List

```json
{
  "type": "bulletList",
  "content": [
    { "type": "listItem", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Item" }] }] }
  ]
}
```

### Ordered List

```json
{
  "type": "orderedList",
  "content": [
    { "type": "listItem", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "First" }] }] }
  ]
}
```

### Code Block

```json
{ "type": "codeBlock", "attrs": { "language": "python" }, "content": [{ "type": "text", "text": "print('hello')" }] }
```

### Blockquote

```json
{ "type": "blockquote", "content": [{ "type": "paragraph", "content": [{ "type": "text", "text": "Quoted text" }] }] }
```

### Horizontal Rule

```json
{ "type": "rule" }
```

## Inline Nodes

### Text (with optional marks)

```json
{ "type": "text", "text": "Hello", "marks": [{ "type": "strong" }] }
```

### Hard Break (line break within a paragraph)

```json
{ "type": "hardBreak" }
```

### Mention

```json
{ "type": "mention", "attrs": { "id": "account-id", "text": "@User Name" } }
```

### Inline Card (link preview)

```json
{ "type": "inlineCard", "attrs": { "url": "https://example.com" } }
```

### Emoji

```json
{ "type": "emoji", "attrs": { "shortName": ":thumbsup:" } }
```

### Status Badge

```json
{ "type": "status", "attrs": { "text": "IN PROGRESS", "color": "blue" } }
```

## Marks (Text Formatting)

Apply to text nodes via the `marks` array. Multiple marks can be combined.

| Mark | JSON | Renders as |
|------|------|------------|
| Bold | `{"type": "strong"}` | **bold** |
| Italic | `{"type": "em"}` | *italic* |
| Underline | `{"type": "underline"}` | underlined |
| Strikethrough | `{"type": "strike"}` | ~~strikethrough~~ |
| Inline code | `{"type": "code"}` | `code` |
| Link | `{"type": "link", "attrs": {"href": "https://..."}}` | clickable link |
| Text color | `{"type": "textColor", "attrs": {"color": "#ff0000"}}` | colored text |
| Superscript | `{"type": "subsup", "attrs": {"type": "sup"}}` | superscript |
| Subscript | `{"type": "subsup", "attrs": {"type": "sub"}}` | subscript |

### Combining Marks

```json
{ "type": "text", "text": "bold and italic", "marks": [{ "type": "strong" }, { "type": "em" }] }
```

## Common Patterns

### Bold label followed by normal text

```json
{
  "type": "paragraph",
  "content": [
    { "type": "text", "text": "Status: ", "marks": [{ "type": "strong" }] },
    { "type": "text", "text": "Complete" }
  ]
}
```

### Link

```json
{
  "type": "paragraph",
  "content": [
    { "type": "text", "text": "See the " },
    { "type": "text", "text": "documentation", "marks": [{ "type": "link", "attrs": { "href": "https://example.com" } }] }
  ]
}
```
