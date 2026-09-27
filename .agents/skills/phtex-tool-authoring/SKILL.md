---
name: phtex-tool-authoring
description: >-
  Comprehensive protocol, engineering standards, and procedural instructions for authoring new tool modules
  in the PhTex developer encyclopedia. Use this skill whenever creating, expanding, or updating tool
  documentation (such as neovim, fzf, ripgrep, jq, git, docker, zsh) in PhTex with bilingual (en/fa) Markdown
  and modern light-theme vector PDFs.
---

# PhTex Tool Authoring Specification & Protocol

This skill dictates the exact protocol for synthesizing new tool dossiers for the **PhTex** developer tools reference. Every tool module must adhere to strict structural, linguistic, typographic, and compilation standards.

---

## 1. Directory Structure Standard

Every tool must be implemented concurrently across both English (`en/`) and Persian (`fa/`) directories.

```text
PhTex/
├── README.md                      # Global registry (must be updated with new tool entry)
├── en/
│   └── <tool_name>/
│       ├── <tool_name>.md         # Comprehensive English Markdown manual
│       ├── <tool_name>.html       # Standalone Modern Light-Theme HTML source
│       └── <tool_name>.pdf        # Headless-compiled vector PDF
└── fa/
    └── <tool_name>/
        ├── <tool_name>.md         # مستندات تخصصی و جامع به زبان فارسی
        ├── <tool_name>.html       # قالب مدرن تم روشن با ساختار کامل RTL
        └── <tool_name>.pdf        # پی‌دی‌اف وکتور تولیدشده با Edge
```

---

## 2. Markdown Manual Standard (`<tool>.md`)

Both language editions must contain an uncompromising, production-grade technical manual with zero conversational filler:

1. **Document Metadata & Header:**
   - Document Class, Technical Domain, Target Tool Version.
2. **Architectural Anatomy & Topology:**
   - Formal description of internals, data flow, memory model, and operating system interaction.
   - ASCII / Unicode flow diagram depicting component decoupling.
3. **Control Matrix & Command Grammar:**
   - Structured comparison tables: Action vs. CLI Command vs. Interactive Chord/Hotkey vs. Execution Logic.
4. **Specialized Engine / Subsystem Analysis:**
   - Tool-specific operational deep dives (e.g., regex engine for `ripgrep`, AST parsing for `jq`, buffer/undo trees for `neovim`, index/DAG for `git`).
5. **Hardened Production Configuration:**
   - Complete, copy-paste ready config snippet (`.<tool>rc`, `.conf`, or `.lua`) with line-by-line commentary explaining performance rationales.
6. **Automated Workflow Routine:**
   - Executable, non-interactive POSIX bash script illustrating automation or workspace orchestration.
7. **Diagnostic Matrix & Remediation:**
   - Table detailing root causes and definitive fixes for common edge cases, errors, and terminal compatibility pitfalls.

---

## 3. Persian Localization Protocol (`fa/<tool>/<tool>.md`)

* **Technical Precision:** Use authentic, professional Iranian computer engineering vocabulary (e.g., نشست for Session, پنجره for Window, پنل for Pane, بافر for Buffer, سوکت یونیکس for Unix Domain Socket).
* **Directional Integrity:** All English identifiers, CLI commands, regex patterns, key combinations, and code blocks must retain LTR formatting (`direction: ltr; text-align: left;`).
* **Punctuation:** Use Persian numerals and punctuation where appropriate in prose, while retaining ASCII digits in code blocks.

---

## 4. Modern Light-Theme HTML & PDF Design System

PDFs must be generated from an HTML template strictly adhering to the **PhTex Light Design System**.

### CSS Variables & Typography
```css
/* Core Palette */
--canvas: #ffffff;
--surface: #f8fafc;
--border: #e2e8f0;
--text-primary: #0f172a;
--text-secondary: #475569;
--text-muted: #64748b;
--accent-indigo: #4f46e5;
--accent-cyan: #0284c7;
--accent-emerald: #059669;

/* Page Constraints */
@page {
  size: A4 portrait;
  margin: 10mm 10mm 12mm 10mm;
}

* {
  -webkit-print-color-adjust: exact !important;
  print-color-adjust: exact !important;
}
```

### Layout Elements
1. **Hero Banner:** Linear gradient (`#f8fafc` ➔ `#eef2ff` ➔ `#e0e7ff`), border `#c7d2fe`, pill badges (`.badge`, `.badge.cyan`, `.badge.green`), bold title with accent text.
2. **Architectural Flow Card:** Flexbox pipeline connecting system nodes (`.arch-node` in white `#ffffff`, border `#cbd5e1`, with `.highlight` in `#eef2ff` / `#818cf8`).
3. **Structured Tables:** Clean zebra striping (`#f8fafc`), `#f1f5f9` header with `#cbd5e1` borders.
4. **Keyboards (`<kbd>`):** Styled box with subtle 3D bottom border:
   `background: #f8fafc; border: 1px solid #cbd5e1; border-bottom: 2px solid #94a3b8; font-weight: 700;`
5. **Code Blocks (`<pre>`):** `#f8fafc` surface, `#e2e8f0` border, styled tokens (`.keyword: #4338ca`, `.string: #047857`, `.comment: #64748b; font-style: italic;`).
6. **Persian HTML Template Specifics:**
   - Root `<html lang="fa" dir="rtl">`
   - `font-family: 'Vazirmatn', -apple-system, BlinkMacSystemFont, "Segoe UI", Tahoma, Arial, sans-serif;`
   - `direction: rtl; text-align: right;`
   - All code blocks (`pre`, `code`, `kbd`) set to `direction: ltr; text-align: left; display: inline-block;`.
   - Reverse the architectural flow arrows: `flex-direction: row-reverse`.

---

## 5. Automated PDF Compilation Pipeline

Use Microsoft Edge or Chrome in headless mode with high-fidelity rasterization flags:

### PowerShell Invocation:
```powershell
$edge = "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
Start-Process -FilePath $edge -ArgumentList `
  "--headless=new", `
  "--disable-gpu", `
  "--no-pdf-header-footer", `
  "--print-to-pdf=`"C:\path\to\<tool>.pdf`"", `
  "`"C:\path\to\<tool>.html`"" -Wait -NoNewWindow
```

### Verification Criteria:
* PDF length must exceed 100,000 bytes.
* Page count must balance into cleanly paginated 2-page or 4-page spreads using `<div class="page-break"></div>`.

---

## 6. Registry Synchronisation Protocol

Whenever a new tool is authored:
1. Update `PhTex/README.md`:
   - Add tool to the `Tool Index / فهرست ابزارها` markdown table with direct links to both `en/` and `fa/` documents.
   - Mark the item as checked `- [x]` in the `## Roadmap` section.
2. Commit changes with a conventional semantic message:
   `git add . && git commit -m "feat(<tool>): add bilingual manual and light-theme PDF dossier"`
3. Push to remote:
   `git push origin main`
