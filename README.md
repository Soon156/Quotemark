# Annotator

**A single-file, zero-dependency, zero-install text annotator.**
Select text → tag it or write a note → export one structured Markdown block and
paste it back to an LLM as revision instructions.

> Your source file is never touched. The tool only reads.

## Use it

Download [`annotator.html`](annotator.html) and double-click it. That's the whole setup.

No Node, no build step, no network, no API key. Drop a text file in and start marking.

## What it's for

Telling an LLM "which line, what's wrong, how to fix it" out loud drops items,
smears line numbers, and invites improvisation. Writing that list by hand is slow.
This turns producing the list into a selection gesture.

The export looks like this:

```markdown
# Notes on ch1.md - 2 total

Work through these one at a time. Do not merge them and do not improvise.
Report the final line number for each change.

## [1] L42 · Chapter 3 · Delete
Source:
~~~
He knew, in that moment, that the silence weighed more than any words could.
~~~
Rule: Cut the whole passage. Do not write a replacement.

## [2] L57-58 · Chapter 3 · Telling
Source:
~~~
She felt a powerful, almost overwhelming fear.
~~~
Note: replace with a physical reaction
Rule: States the emotion instead of showing it. Replace with a concrete action or physical detail.
```

## Features

- Any plain text file (`.md` `.txt` `.py` `.srt` `.log` …); picks mono or serif by extension
- Tag it, write a note, or both — **either one alone is enough**
- Sidebar cards: filter by tag, jump, edit, step through with `F8`
- Outline: Markdown headings, horizontal rules, or **a custom regex** — works for code, subtitles, and logs too
- Tags are editable: name, color, order, and a "rule" that travels with the note into the export
- Four built-in presets: Generic · Prose editing · Code review · Translation review
- Settings export/import as JSON (**never includes notes**)
- Three resizable panes; widths are remembered
- IME-safe input, keyboard shortcuts, in-page dialogs that browsers can't suppress

## Keyboard

| Key | Action |
| --- | --- |
| `Ctrl` `/` | New note on the current selection |
| `Ctrl` `Enter` | Submit |
| `F8` / `Shift` `F8` | Next / previous note |
| `Ctrl` `Shift` `I` | Export |
| `Ctrl` `B` | Toggle outline |
| `Esc` | Close the topmost layer |

## Where notes are stored

In the browser's `localStorage`, keyed by filename + length + content hash.
**Never written into your source file.**

A different browser, a private window, or clearing site data loses them.
**The only reliable save is the export.** Export as soon as you're done.

The tool probes `localStorage` on startup and shows an orange warning in the
header if it isn't available.

## Languages

English is hardcoded and is always the fallback. Other languages live in
[`i18n/`](i18n/) — one folder per language code, containing the interface strings,
the tag presets, and the export wording.

On first run the interface follows `navigator.language`; if there's no matching
folder it stays in English. Your choice after that is remembered.

**Language packs need the repository**, not just `annotator.html` — the single
file on its own is English only.

Adding a language means copying `i18n/_template/` and translating four files.
No code. PRs welcome.

## Profiles

[`profiles/`](profiles/) holds complete shareable setups — tag table plus outline
rule, export note, and display settings. Import one under ⚙ → Import from file.

Presets and profiles are different things: a **preset** is just a tag table that
ships with a language pack, while a **profile** is a whole configuration you can
hand to someone else.

## Roadmap

See [PLAN.md](PLAN.md). Next up is **re-anchoring** — right now editing one
character in the source invalidates every cached note.

## License

MIT
