---
name: phtex-tool-authoring
description: >-
  Comprehensive protocol, engineering standards, and procedural instructions for authoring new tool modules
  in the HpTex developer encyclopedia. Use this skill whenever creating, expanding, or updating tool
  documentation (such as neovim, fzf, ripgrep, jq, git, docker, zsh) in HpTex with bilingual (en/fa) Markdown
  and modern vector PDFs.
---

# HpTex Tool Authoring Specification & Protocol

This skill dictates the exact protocol for synthesizing new tool dossiers for the **HpTex** developer tools reference. Every tool module must adhere to strict structural, linguistic, typographic, and compilation standards.

---

## 1. Directory Structure Standard: `HpTex/<Tool>/<Language>/<Files>`

Every tool must be implemented under its own dedicated root tool directory, partitioned by language (`en/` and `fa/`):

```text
HpTex/
├── README.md                      # Global registry (must be updated with new tool entry)
└── <tool_name>/
    ├── en/
    │   ├── <tool_name>.md         # Comprehensive English Markdown manual
    │   ├── <tool_name>.html       # Standalone Modern Light/Dark HTML source
    │   └── <tool_name>.pdf        # Headless-compiled vector PDF
    └── fa/
        ├── <tool_name>.md         # مستندات تخصصی و جامع به زبان فارسی
        ├── <tool_name>.html       # قالب مدرن تم لایت/دارک با ساختار کامل RTL
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
   - Tool-specific operational deep dives (e.g., regex engine for `ripgrep`, AST parsing for `jq`, buffer/undo trees for `neovim`, index/DAG for `git`, MediaCodec for `scrcpy`).
5. **Hardened Production Configuration:**
   - Complete, copy-paste ready config snippet (`.<tool>rc`, `.conf`, or `.py`) with line-by-line commentary explaining performance rationales.
6. **Automated Workflow Routine:**
   - Executable, non-interactive POSIX bash script illustrating automation or workspace orchestration.
7. **Diagnostic Matrix & Remediation:**
   - Table detailing root causes and definitive fixes for common edge cases, errors, and terminal compatibility pitfalls.

---

## 3. Persian Localization Protocol (`<tool>/fa/<tool>.md`)

* **Technical Precision:** Use authentic, professional Iranian computer engineering vocabulary (e.g., نشست for Session, پنجره for Window, پنل for Pane, بافر for Buffer, سوکت یونیکس for Unix Domain Socket).
* **Directional Integrity:** All English identifiers, CLI commands, regex patterns, key combinations, and code blocks must retain LTR formatting (`direction: ltr; text-align: left;`).
* **Punctuation:** Use Persian numerals and punctuation where appropriate in prose, while retaining ASCII digits in code blocks.

---

## 4. Modern HTML & PDF Design System with Theme Switcher

HTML sources must feature a dynamic light/dark theme switcher using CSS custom properties (`:root` and `[data-theme="dark"]`).

### CSS Variables & Typography
```css
:root {
  --canvas: #ffffff;
  --surface: #f8fafc;
  --border: #e2e8f0;
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #64748b;
  --accent-indigo: #4f46e5;
  --accent-cyan: #0284c7;
  --accent-emerald: #059669;
}

[data-theme="dark"] {
  --canvas: #0b0f19;
  --surface: #111827;
  --border: #1f2937;
  --text-primary: #f8fafc;
  --text-secondary: #cbd5e1;
  --text-muted: #94a3b8;
  --accent-indigo: #818cf8;
  --accent-cyan: #38bdf8;
  --accent-emerald: #34d399;
}

/* Strict Invariant: Forced Light Theme for PDF Printing */
@media print {
  :root, [data-theme="dark"] {
    --canvas: #ffffff !important;
    --text-primary: #0f172a !important;
    --border: #e2e8f0 !important;
  }
  .theme-toggle-btn {
    display: none !important;
  }
}
```

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
  "--print-to-pdf=`"C:\path\to\<tool>\<lang>\<tool>.pdf`"", `
  "`"C:\path\to\<tool>\<lang>\<tool>.html`"" -Wait -NoNewWindow
```

### Verification Criteria:
* PDF length must exceed 100,000 bytes.
* Page count must balance into cleanly paginated 2-page spreads using `<div class="page-break"></div>`.

---

## 6. Registry Synchronisation Protocol

Whenever a new tool is authored:
1. Update `HpTex/README.md`:
   - Add tool to the `Tool Index / فهرست ابزارها` markdown table with direct links to `<tool>/en/<tool>.md` and `<tool>/fa/<tool>.md`.
   - Mark the item as checked `- [x]` in the `## Roadmap` section.
2. Commit changes with a conventional semantic message:
   `git add . && git commit -m "feat(<tool>): add bilingual manual and vector PDF dossier"`
3. Push to remote:
   `git push origin main`
