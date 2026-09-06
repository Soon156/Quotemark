# Profiles

A **profile** is a complete, shareable setup: tag table plus outline rule, export
note, and display settings. Import one under ⚙ → "Import from file…".

This is different from a **preset**. A preset is only a tag table, and presets
ship with the app (and with each language pack under `i18n/<code>/tags.js`). A
profile is the whole configuration, so it is worth publishing only when it adds
something a preset cannot — a custom outline regex, a font, a different export
note.

Profiles never contain notes. They are safe to share.

## Contributing one

Submit a JSON file. No code required.

```json
{
  "v": 1,
  "name": "Short descriptive name",
  "tags": [{ "k": "Tag", "c": "#4a7ba7", "rule": "What to do about it" }],
  "tocRule": "all | h2 | h3 | hr | custom",
  "tocCustom": "regex, first capture group becomes the title",
  "exportNote": "Instruction line placed at the top of the export",
  "docFont": "auto | serif | sans | mono",
  "docSize": "14px | 16px | 18px | 20px"
}
```

Name the file after what it is for, not what language it is in.

## Current profiles

| File | For |
| --- | --- |
| `code-review.json` | Source files. Outlines by function/class/type declaration, monospace. |
| `prose-zh.json` | Revising Chinese web fiction. Moves into `i18n/zh-CN/tags.js` once that language pack lands. |
