# Annotator — roadmap and design decisions

> Supersedes `ANNOTATOR_PLAN.md` in the `kokoro-connect` repository. That plan's
> backend, CodeMirror 6, Fastify, and in-app AI calls are all cancelled — see §3.

Last updated: 2026-09-06

---

## 1. What this is

A **single-file, zero-dependency, zero-install** text annotator.

Open any plain text file → select → tag it or write a note → export one
structured Markdown block and paste it into an LLM conversation as revision
instructions.

**The source file is never modified.** The tool only ever reads it.

### The problem

Telling an LLM "which line, what's wrong, how to fix it" out loud drops items,
smears line numbers, and invites improvisation. Writing the list by hand is slow.
This turns producing the list into a selection gesture.

### What it is not

Not an editor, not an AI client, not a collaboration platform.
**The exported text is the product.**

---

## 2. Current state

`annotator.html`, ~1300 lines, single file, no build step.

| Area | Status |
| --- | --- |
| File open (drop / picker) | ✅ any plain text, not just Markdown |
| Line numbers, selection → `(line, col)` anchors | ✅ survives text nodes split by `<mark>` |
| Notes: tag + body, either alone is enough | ✅ |
| Sidebar cards, tag filter, prev/next navigation | ✅ |
| Outline (headings / rules / custom regex) | ✅ with scroll-spy highlight |
| Tag management (add, delete, reorder, color, rule text) | ✅ |
| Four built-in presets | ✅ Generic / Prose / Code review / Translation |
| Settings import & export (JSON) | ✅ never includes notes |
| Markdown export + clipboard | ✅ |
| `localStorage` cache + availability probe | ✅ orange header warning when unavailable |
| Three resizable panes | ✅ widths persisted |
| In-page dialogs (no native `confirm()`) | ✅ including type-to-confirm |
| IME (CJK input) guarding | ✅ |

### Known defects

- **Overlapping highlights**: when two notes overlap, `paint()` clamps the second
  one. The data is correct, it just isn't drawn.
- **Fragile anchors**: edit one character in the source and the whole cache is
  invalidated, because it's keyed by content hash. See M1.
- **Interface strings are still Chinese** — being extracted now, see M0.

### Verified environment facts

Measured on the author's machine (Firefox 155, Windows 11, `file://`):

- `localStorage` **works** on a `file://` origin — the cache genuinely survives refresh.
- Classic `<script src="i18n/…">` **loads** from `file://`. This is what the
  language-pack design rests on.
- `fetch()` fails on `file://` (`TypeError`, null origin) — never rely on it.
- **Firefox does not implement the File System Access API at all.** M5's
  "localhost + direct file read/write" plan does not work there; it needs Chrome
  or Edge, or the writing has to move server-side.

### Design decisions (do not walk these back)

1. **One file.** External JS for the *app itself* would break `file://`,
   zero-install, and zero-build all at once. Language packs are the one
   exception, and they degrade to English when absent.
2. **No AI calls in the app.** No API key, no network, works offline. This is a
   differentiator, not laziness.
3. **Never write the source file.** Notes live in the browser; the export goes
   through the clipboard.
4. **Configuration is data, not code.** Users supply declarative values (a regex,
   a template string, JSON) — never a function.
5. **English is hardcoded and is always the fallback.** Every other language is a
   folder under `i18n/`.

---

## 3. Prior art (surveyed 2026-09)

| Project | Size | Focus | Relationship |
| --- | --- | --- | --- |
| [plannotator](https://github.com/backnotprop/plannotator) | 8.5k★ | Annotate code diffs / agent plans, one-click back to the agent | The leader in this lane. Code-oriented; needs a CLI or VS Code extension. Prose, and non-English prose, are not its target |
| [md-annotator](https://github.com/konradmichalik/md-annotator) | 6★ | **Nearly the same concept**: local server + browser, annotate Markdown, export and re-import, ships a Claude Code plugin | The closest comparison. More features (multi-file, mermaid, KaTeX, undo/redo, insertions) but requires Node |
| Obsidian plugins | — | Sidebar comments, source untouched, Markdown export | [Side Comments](https://community.obsidian.md/plugins/side-comments) / [Tandem Comments](https://community.obsidian.md/plugins/tandem-comments) / [Siden](https://community.obsidian.md/plugins/siden) / [Document Comments](https://community.obsidian.md/plugins/document-comments). Well made, but you have to live in Obsidian |
| [markup](https://github.com/samueldobbie/markup) | — | Web document annotation (NLP entity labelling) | Different goal |
| pdfannots / remarks / Highlights | — | PDF & EPUB highlight extraction | Crowded; explicitly out of scope |

**Conclusion:** the need is real but niche (the closest equivalent has 6 stars).
Being first is not a moat.

### The only differentiator: friction

| | What you install |
| --- | --- |
| plannotator | A CLI or VS Code extension |
| md-annotator | Node |
| Obsidian plugins | Obsidian |
| **This** | **Double-click one .html** |

Plus: offline with no API key, and outline rules that work on non-Markdown files.

**Zero-install is the entire pitch. Any proposal that breaks it gets rejected.**

### Two standards to adopt rather than reinvent

1. **[W3C Web Annotation Data Model](https://www.w3.org/TR/annotation-model/)'s `TextQuoteSelector`**
   — `prefix` / `exact` / `suffix`, exactly what re-anchoring needs.
   [Apache Annotator](https://annotator.apache.org/docs/api/modules/dom.html) is a
   reference implementation, down to returning a generator when a selector matches
   in several places. Adopting it also makes the sidecar JSON interoperable with
   Hypothesis and friends. **Do not invent an anchor format.**

2. **[CriticMarkup](https://github.com/CriticMarkup/CriticMarkup-toolkit)**
   — plain-text editorial syntax (`{--delete--}` `{++insert++}` `{==highlight==}{>>note<<}`)
   understood by Marked, MultiMarkdown, pandoc, and mkdocs. One extra export
   format buys the whole toolchain.

---

## 4. Milestones

### M0 — English baseline + i18n ⬅ **in progress**

The repository ships in English; everything else is a language pack.

**Step 1 — English baseline** ✅ done
Code comments, README, PLAN, built-in presets, export wording.

**Step 2 — i18n machinery**
- ~130 interface strings extracted to keys; English stays hardcoded as the fallback
- HTML carries `data-i18n` / `data-i18n-title` / `data-i18n-ph`; JS uses `t(key, vars)`
- Language switch injects `<script src="i18n/<code>/*.js">`; a missing file falls
  back to English for that category only, so a half-finished translation still works
- First run follows `navigator.language`, then remembers the user's choice
- `tools/gen-i18n-template.js` regenerates `i18n/_template/` from the hardcoded English

**Step 3 — first language pack**: `i18n/zh-CN/` with all four files, verified end to end.

### M1 — Re-anchoring ⬅ **blocks everything after it**

Edit one character in the source and every note is invalidated today. Until this
is fixed, auto-reload, writing back to files, and multi-file are all pointless.

- Anchors become W3C three-part: `{ exact, prefix, suffix, start }`
  (line/col stays as the fast path, the triple is the fallback)
- Resolution ladder: exact offset → unique triple match → relaxed prefix/suffix →
  `exact` only with several candidates (ranked by distance from the old position) → fail
- Failures are **never dropped silently**: mark them "unanchored", give them their
  own sidebar section, keep the quoted text so they can be re-attached by hand
- On load, if the content changed, re-anchor before rendering

> ⚠️ Order matters: **re-anchoring must land before auto-reload**, or reloading
> will quietly destroy notes.

### M2 — Round-trip import

The exported Markdown/JSON must load back in. Without it the tool is a one-way
funnel and notes evaporate when a session ends.

- Parse our own Markdown export (`## [n] Lx · section · tag` + `~~~ quote ~~~`)
- JSON round-trips losslessly, anchors included
- On import, re-anchor against the current file; conflicts go to M1's unanchored section

### M3 — Export templates + profiles

**The export is the product**, so its shape should not be hardcoded.

- **Template**: three strings (head / item / tail) with `{{placeholders}}` and one
  rule — *drop any line whose placeholders are all empty*. No template language
  (handlebars in a single file is pure liability); roughly 30 lines.
  Placeholders: `{{n}} {{loc}} {{line}} {{sec}} {{tag}} {{rule}} {{quote}} {{body}} {{file}} {{total}} {{date}}`
  Built-ins: default / compact (one line per note) / with context / custom.
  The existing `EXPORT` object is already shaped for this.
- **Profiles**: `tags + tocRule + tocCustom + exportTemplate + exportNote + docFont + docSize`
  as one named JSON. `profiles/` accepts PRs — **contributing one is submitting a
  JSON file, not writing code**. That is the only realistic form of ecosystem at
  this size.

### M4 — Polish to publishable

- Note state: handled / unhandled, and export only the unhandled ones
- In-document search (`Ctrl+F`)
- Fix the overlapping-highlight clamp

### M5 — The `npx` tier (optional)

Only needed when direct file read/write and change-watching matter.

Key fact: **`localhost` is a secure context, `file://` is not.** On localhost the
File System Access API becomes available → open the real file, write
`<name>.anno.json` beside it, watch for changes. **That is far enough; no native
shell required.**

- A minimal static-serving CLI: `npx @kokseng/annotator ch1.md`
- Sidecar `<filename>.anno.json`
- Multi-file / folder mode

⚠️ **Firefox has no File System Access API.** On Firefox this tier degrades to
"served from localhost" without direct writes, or the server has to do the writing.

### M6 — Tauri (probably never)

Worth it only for a dock icon, `.md` file associations, and auto-update.
Functionally it buys nothing over M5.

If it ever happens, Tauri not Electron: the app is one HTML file with no Node
dependency, so there is no Rust backend to write and the learning cost is near zero.
([Comparison](https://www.pkgpulse.com/guides/electron-vs-tauri-2026): 3–10 MB vs
120–200 MB bundles, 380 ms vs 1420 ms cold start, 42 MB vs 168 MB resident — from a
review blog, not measured here.)

**Electron has no argument in this scenario.**

---

## 5. Explicitly out of scope

- ❌ **Plugin architecture / external JS for the app** — breaks the single file, and
  a project this size will not grow a plugin ecosystem
- ❌ **In-app AI calls** — if you want AI, feed it the export
- ❌ Collaboration, accounts, a server
- ❌ WYSIWYG editing of the source — this is an annotator, not an editor
- ❌ PDF / EPUB — the ecosystem is saturated (pdfannots, remarks, Highlights)
- ❌ **User-definable note field schemas** — the popover, cards, filter, and import
  would all have to go dynamic for very little gain. Write `severity: high` in the
  body if you need it
- ❌ Configurable shortcuts or theme colors — wait for someone to open an issue

---

## 6. Repository layout

```
annotator/
├── annotator.html      # the app. one file, no dependencies, double-click to run
├── index.html          # GitHub Pages entry (redirects to annotator.html)
├── README.md
├── PLAN.md             # this file
├── LICENSE             # MIT
├── .gitignore
├── i18n/               # one folder per language; English is hardcoded in the app
│   ├── README.md
│   ├── _template/      # copy this to start a language
│   │   ├── meta.js
│   │   ├── ui.js
│   │   ├── tags.js
│   │   └── export.js
│   └── zh-CN/
│       └── …
├── profiles/           # complete shareable setups, PR target
└── samples/
    └── demo.md         # open it and start clicking
```

Everything except `annotator.html` is packaging. **The app must always be
downloadable and runnable on its own** — in English, without the rest of the repo.

### Migration note

The app is still tracked by `Soon156/kokoro-connect`, alongside an entire fan
novel (`ch1.md`–`ch5.md`, `references/`, the epub) and its setting bible.

**That repository must not simply be made public.** This is a clean `git init`,
not a `filter-branch` of the old one — its single `first commit` carries the whole
novel in history. The stale copy should be deleted from `kokoro-connect` once this
repository is live.

---

## 7. Open questions

- **License**: MIT for now. Change it before pushing if Apache-2.0 or a NOTICE is wanted.
- **Repository and npm package name**: `annotator` is too generic to find. Needs a real name.
- **"Split view"** (raised, never settled): A = render one section at a time;
  B = side-by-side on two locations. Neither is scheduled.
