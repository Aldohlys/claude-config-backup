---
name: reference_markdown_pdf_pipeline
description: How to render workspace markdown docs to HTML + PDF (pandoc → Chrome headless) on this Windows machine
metadata: 
  node_type: memory
  type: reference
  originSessionId: 42c93396-2b7d-4983-a6b2-7e9c39f3bf2e
---

Render a `.md` doc to PDF in this workspace via **pandoc → standalone HTML → Chrome headless print-to-pdf**. No wkhtmltopdf/weasyprint installed; pandoc + Chrome are. (This is the same toolchain that produced the existing `*_Checklist.html`.)

**Tool locations (Git-Bash paths):**
- pandoc: `/c/Users/aldoh/AppData/Local/Pandoc/pandoc` (v2.9; not on PATH by default)
- Chrome: `/c/Program Files/Google/Chrome/Application/chrome.exe` (Edge also present)

**Step 1 — md → standalone HTML** (embed a small print CSS for bordered tables + monospace ASCII blocks; `-f gfm` so GitHub-flavored tables + `\$` escapes render literally, not as math):
```
pandoc "$base.md" -f gfm -t html5 --standalone --embed-resources --css bot.css \
  --metadata title="$base" -o "$base.html"
```

**Step 2 — HTML → PDF** (use a temp `--user-data-dir` to avoid clashing with a running Chrome; `--no-pdf-header-footer` drops the default URL/date chrome; `file:///` needs the absolute Windows path):
```
chrome.exe --headless=new --disable-gpu --no-pdf-header-footer \
  --user-data-dir="<temp>/chromeprofile" \
  --print-to-pdf="<DIR>/$base.pdf" "file:///<DIR>/$base.html"
```

CSS that worked well: `pre{white-space:pre}` (keeps the ASCII quick-ref card box intact), `table{border-collapse:collapse}` with `th/td` borders, `@media print{table{page-break-inside:avoid}}`.

**Gotchas:** Chrome writes compressed object streams, so a `grep "/Type/Page"` page-count check returns 0 even on a valid PDF — verify by the `%PDF` header + nonzero size instead. Validate `\$` rendered literally by grepping the HTML for the line (e.g. `<$10` should appear as a plain `$`, not inside a math span).

**Repo tracking (see [[project_newtrading_repo_policy]]):** `.md` and `.pdf` are tracked; `.html` is gitignored — so commit the md + pdf, the html regenerates on demand. Used 2026-06-25 to ship `BOT_Comprehensive_Checklist` + `BOT_Quick_Reference` as md+pdf. See [[project_bot_trading_plan_review]].
