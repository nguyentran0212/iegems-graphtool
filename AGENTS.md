# agent.md

Instructions for a coding agent working on this repository.

## Project at a glance

This is the **IEGems Graph Tool** — a static GraphTool instance with no build step. It is forked from [Rohsa's GraphTool](https://rohsa.gitlab.io/graphtool/), which is itself a fork of [CrinGraph](https://github.com/mlochbaum/CrinGraph) by Marshall Lochbaum, rebranded for the [IEGems](https://iegems.nk-tran.com) IEM review blog. The entry point is `index.html`; the fork's config is `config_iegems.js`; the fork's data and registry live in `data_iegems/`. Each new IEM follows a short cycle: **drop data → register → preview → commit → push**.

## License and attribution — never remove

- `LICENSE` — permissive upstream license by Marshall Lochbaum. Do not delete, rename, or reformat.
- The **upstream paragraphs of `README.md`** (Rohsa modifications, CrinGraph credits, sample-data notes, contact section). Only the small **IEGems header block** at the very top of `README.md` is yours; everything that follows is preserved attribution.
- `Configuring.md`, `Documentation.md` — upstream reference docs. Leave intact.
- `cringraph-favicon.png`, `cringraph-icon.png`, `cringraph-icon_tr.png`, `cringraph-logo.svg`, `favicon.ijs`, `favicon.png` — upstream branding and favicon assets. Leave intact.
- Do not rewrite git history to strip upstream authorship.

## Repo map

| Path | Role | Edit? |
|---|---|---|
| `index.html` | Entry point (loads CDN scripts, `config_iegems.js`, `graphtool.js`) | Rarely — only favicon / meta tweaks |
| `config_iegems.js` | Fork config (branding, header, targets, default options) | Yes, for branding / target changes |
| `data_iegems/` | Fork data folder + `phone_book.json` registry | Yes, for new IEMs |
| `graphtool.js` | CrinGraph rendering engine | No |
| `equalizer.js` | Parametric EQ feature | No |
| `saveSvgAsPng.js` | PNG export helper | No |
| `graphAnalytics.js` | GA4 measurement loader | No |
| `style.css`, `style-alt.css`, `style-alt-theme.css`, `styles/` | Stylesheets (loaded by `setLayout()` in `config_iegems.js`) | No |
| `LICENSE`, `Configuring.md`, `Documentation.md` | Upstream reference & license | No |
| `cringraph-*`, `favicon.*` | Branding | No |
| `README.md` | Fork header block (top) is yours; rest is preserved upstream | Edit only the IEGems block |
| `AGENTS.md` | This file | Edit in-place if scope changes |

## Adding a new IEM

This is the only workflow an agent needs to know.

### 1. Drop the data

Place `<FileBase> L.txt` and `<FileBase> R.txt` in `data_iegems/`.

REW export format: two whitespace / comma / semicolon-separated columns `Freq(Hz) SPL(dB)`. Lines starting with `*` are comments and are ignored by `tsvParse()` in `config_iegems.js`. Channel suffixes (`L` / `R`) come from the `default_channels` config in `config_iegems.js`; the loader reads each file independently.

### 2. Validate JSON before staging

After any edit to `phone_book.json`:

```
python3 -m json.tool data_iegems/phone_book.json > /dev/null
```

The file must remain valid JSON. **Do not reformat the rest of the file** — match the existing 2-space indentation of the surrounding block, and only insert or edit the lines you need.

### 3. Edit `data_iegems/phone_book.json`

Find the brand by **exact name (case-sensitive)**. If a block already exists for the manufacturer, append your new entry to its `phones` array. If no block exists, add a new `{ "name": "<Manufacturer>", "phones": [...] }` block in the right alphabetic-ish position.

**Never create a duplicate brand block.** CrinGraph matches the brand key by exact string, so duplicates produce duplicated brand groups in the sidebar. There used to be a duplicate `"Sony"` block — that has been consolidated; do not reintroduce the pattern.

### 4. Pick the entry shape

| Situation | Format |
|---|---|
| One measurement, display name == file base | `{ "name": "Advar" }` |
| Multiple variants (tuning switches, sample units, mods, …) | `{ "name": "FH15", "file": ["FH15 Balanced", "FH15 Bass", "FH15 Treble"] }` |
| Earbud | `{ "name": "FF3 (Earbuds)", "file": ["FF3"] }` — display gets the `(Earbuds)` suffix, the file basename is clean |
| Display name differs from filename | `{ "name": "Canon 2", "file": ["Canon 2 00", "Canon 2 01", "Canon 2 10", "Canon 2 11"] }` |

Every variant in `file` must exist on disk as `<Variant> L.txt` / `<Variant> R.txt`.

### 5. DIY builds

Custom / hand-built IEMs go under the existing `"DIY"` or `"IE-Gems DIY"` blocks. Do not create a third DIY block.

## Local preview

From the repo root:

```
python3 -m http.server 8080 --bind 127.0.0.1
```

Then open `http://127.0.0.1:8080/`. Verify the new IEM appears under its manufacturer in the left sidebar and that both channels render correctly (use the wishbone-shaped channel selector to switch between L, R, and average).

To stop the server:

```
kill $(lsof -t -i :8080)
```

If 8080 is taken, pick another free port and adjust the URL. The server binds to `127.0.0.1` only — there is no remote access by default.

## Targets

The `targets` array near the top of `config_iegems.js` is the source of truth. Each filename corresponds to a single `<Name>.txt` curve in `data_iegems/` (targets do not have L/R — they are single curves). Current groupings:

```
{ type: "Preference", files: ["IEGems"] },
{ type: "Reference",  files: ["IEF IEM", "Diffuse Field", "Harman IE 2019 v2"] },
{ type: "Reviewer",   files: ["Precog", "Super Review", "Toranku", "Vamp898"] }
```

To add a new reviewer target, drop the `.txt` in `data_iegems/` and add its basename to the appropriate `type` row. Validate `config_iegems.js` is still valid JS (no obvious syntax errors around your edit) before previewing.

## Commit and push

- Stage **only** the files you changed:
  ```
  git add data_iegems/Performer8s\ L.txt data_iegems/Performer8s\ R.txt data_iegems/phone_book.json
  ```
  Never `git add .` — the repo's history should stay scoped to per-IEM changes, with the rest of the tree untouched across IEM commits.
- Commit message style — **one line, no scope, no body**:
  - Single IEM: `Added Performer8s`
  - Batch: `Added ITO, MS2 Pro, Que, Unicrom, Vulkan 2`
- Push to `origin main`:
  ```
  git push origin main
  ```

## Housekeeping hooks — flag, don't silently fix

While editing `phone_book.json`, watch for these. Surface them to the user instead of taking destructive action.

- **Duplicate brand block** — same `name` key appearing more than once. Ask the user before consolidating into one block.
- **Orphan data file** — a `data_iegems/*.txt` with no matching `phone_book.json` entry, or a JSON entry with no file on disk. Ask whether to register, delete, or leave.
- **Inconsistent JSON formatting** — some brand blocks are pretty-printed across multiple lines, others are condensed. Do **not** reformat globally; only insert or edit the lines you need.

## Do-not-touch list

The following are inherited from CrinGraph / Rohsa or are branding artifacts. Do not edit them without strong reason and explicit user request:

- **Upstream engine**: `graphtool.js`, `equalizer.js`, `saveSvgAsPng.js`, `graphAnalytics.js`.
- **Upstream styles**: `style.css`, `style-alt.css`, `style-alt-theme.css`, `styles/`.
- **Upstream docs and license**: `LICENSE`, `Configuring.md`, `Documentation.md`.
- **Branding assets**: `cringraph-favicon.png`, `cringraph-icon.png`, `cringraph-icon_tr.png`, `cringraph-logo.svg`, `favicon.ijs`, `favicon.png`.
- **Upstream paragraphs of `README.md`** (everything below the small IEGems block at the very top).

And these rules:

- **No build tools.** No `package.json`, no bundler, no TypeScript, no test framework, no preprocessor. This repo is intentionally dependency-free beyond the three CDN scripts at the bottom of `index.html` (d3 v5, d3-selection-multi, fuse.js).
- **No CI additions.** No GitHub Actions workflow, no Jekyll config. Hosting is plain static GitHub Pages.
- **Don't introduce upstream leftovers.** Earlier cleanup already removed `data/`, `data_rohsa/`, `config.js`, `config_hp.js`, `config_rohsa.js`, `graph.html`, `graph_hp.html`, `graph_free.html`, `iframe.html`, and `.jekyll-cache/`. Do not re-add them.

## Out of scope for this file

- **Editing branding / header / default options** — those live in `config_iegems.js` (`page_title`, `headerLogoText`, `headerLogoImgUrl`, `watermark_*`, `init_phones`, the `targets` array, etc.). This `agent.md` mentions them by reference only.
- **CrinGraph feature documentation** — axes, colors, smoothing, normalization, EQ, search, sharing. See `Documentation.md` for those.

## Cross-references

- `README.md` — project overview and upstream attribution
- `LICENSE` — permissive upstream license by Marshall Lochbaum
- `Configuring.md` — CrinGraph configuration keys (reference when editing `config_iegems.js`)
- `Documentation.md` — CrinGraph feature reference (axes, colors, EQ, search, etc.)
