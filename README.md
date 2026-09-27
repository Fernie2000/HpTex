# PhTex — Definitive Developer Tools & Systems Reference

> **High-density, mathematically rigorous, production-grade technical manuals and quick-reference manifests for vital developer utilities.**

[![Language: Multi](https://img.shields.io/badge/Language-English%20%7C%20فارسی-blue.svg)](#index-فهرست)
[![License: MIT](https://img.shields.io/badge/License-MIT-emerald.svg)](LICENSE)
[![Format: Markdown & Modern PDF](https://img.shields.io/badge/Format-MD%20%2B%20PDF-purple.svg)](#available-modules)

---

## Directory Architecture

```text
PhTex/
├── README.md                      # Global entrypoint, tool taxonomy, shortcuts, sources
├── en/                            # English technical manuals and vectors
│   └── tmux/
│       ├── tmux.md                # Exhaustive Markdown reference & production config
│       └── tmux.pdf               # Stylized modern standalone PDF dossier
└── fa/                            # راهنماهای فنی و تخصصی به زبان فارسی
    └── tmux/
        ├── tmux.md                # مستندات جامع، معماری و کانفیگ بهینه‌سازی شده
        └── tmux.pdf               # نسخه سند مدرن و طراحی‌شده‌ی اختصاصی PDF
```

---

## Tool Index / فهرست ابزارها

| Tool | Category | Language | Markdown Dossier | Modern PDF Dossier | Core Capability |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`tmux`** | Terminal Multiplexing | **EN** (English) | [English Guide](en/tmux/tmux.md) | [English PDF](en/tmux/tmux.pdf) | Client-server PTY management, persistent sessions, decoupled multiplexing |
| **`tmux`** | ترمینال مالتی‌پلکسر | **FA** (فارسی) | [راهنمای فارسی](fa/tmux/tmux.md) | [نسخه پی‌دی‌اف فارسی](fa/tmux/tmux.pdf) | مدیریت کلاینت-سرور PTY، سشن‌های پایدار، تقسیم پنل‌ها و بافرها |

---

## Consolidated Shortcut Quick Reference (tmux)

All operations assume standard or optimized Prefix. Default prefix is `Ctrl+b` (`C-b`); hardened systems remap to `Ctrl+a` (`C-a`).

### 1. Session Lifecycle Management

| Operation | CLI Command | Keybinding (Interactive) |
| :--- | :--- | :--- |
| Spawn Named Session | `tmux new-session -s <name>` | `:new -s <name>` |
| List Active Sockets/Sessions | `tmux ls` | `Prefix` + `s` (interactive tree) |
| Attach Target Session | `tmux attach -t <name>` | — |
| Kill Specific Session | `tmux kill-session -t <name>` | `:kill-session` |
| Detach Cleanly (Background) | `tmux detach` | `Prefix` + `d` |
| Rename Session | `tmux rename-session -t <old> <new>` | `Prefix` + `$` |

### 2. Window (Workspace) Navigation

| Operation | CLI / Command Mode | Keybinding |
| :--- | :--- | :--- |
| Create Window | `tmux new-window -n <name>` | `Prefix` + `c` |
| Next / Previous Window | `tmux next-window` / `tmux prev-window` | `Prefix` + `n` / `Prefix` + `p` |
| Jump by Numeric Index | `tmux select-window -t <n>` | `Prefix` + `0..9` |
| Rename Active Window | `tmux rename-window <name>` | `Prefix` + `,` |
| Interactive Window Selector | `tmux choose-tree -w` | `Prefix` + `w` |
| Terminate Window | `tmux kill-window` | `Prefix` + `&` |

### 3. Pane Manipulation & Geometry

| Operation | CLI / Command Mode | Keybinding (Default) | Keybinding (Optimized `.tmux.conf`) |
| :--- | :--- | :--- | :--- |
| Split Horizontally (Top/Bottom) | `tmux split-window -v` | `Prefix` + `"` | `Prefix` + `-` |
| Split Vertically (Left/Right) | `tmux split-window -h` | `Prefix` + `%` | `Prefix` + `\|` |
| Directional Navigation | `tmux select-pane -[U\|D\|L\|R]` | `Prefix` + `Arrow` | `Prefix` + `h/j/k/l` |
| Toggle Zoom (Full-screen Pane) | `tmux resize-pane -Z` | `Prefix` + `z` | `Prefix` + `z` |
| Cycle Pane Layouts | `tmux next-layout` | `Prefix` + `Space` | `Prefix` + `Space` |
| Display Pane Indices & Sizes | `tmux display-panes` | `Prefix` + `q` | `Prefix` + `q` |
| Terminate Active Pane | `tmux kill-pane` | `Prefix` + `x` | `Prefix` + `x` |

### 4. Buffer Operations & Vi Copy Mode

| Operation | Action Sequence |
| :--- | :--- |
| Enter Copy Mode | `Prefix` + `[` |
| Start Selection (Vi mode enabled) | `v` (or `Space` in default emacs mode) |
| Copy Selection to Buffer | `y` (or `Enter` in default emacs mode) |
| Paste Active Buffer | `Prefix` + `]` |
| List Buffer Cache | `tmux list-buffers` or `:list-buffers` |
| Forward Regex Search | `/` |
| Backward Regex Search | `?` |

---

## Curated Sources & Primary Specifications

* **Official Repository & Issue Tracker:** [github.com/tmux/tmux](https://github.com/tmux/tmux)
* **OpenBSD Manual Pages:** [man.openbsd.org/tmux.1](https://man.openbsd.org/tmux.1)
* **Tmux Plugin Manager (TPM):** [github.com/tmux-plugins/tpm](https://github.com/tmux-plugins/tpm)
* **OSC 52 Clipboard Protocol Specification:** [invisible-island.net/xterm/ctlseqs](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html)
* **POSIX Pseudo-Terminal (PTY) Specifications:** [pubs.opengroup.org/onlinepubs/9699919799/basedefs/termios.h.html](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/termios.h.html)

---

## Roadmap

- [x] **tmux** (Terminal Multiplexer) — `en/`, `fa/` [MD + PDF]
- [ ] **neovim** (Modal Hyperextensible Editor) — `en/`, `fa/` [MD + PDF]
- [ ] **fzf** (Command-line Fuzzy Finder) — `en/`, `fa/` [MD + PDF]
- [ ] **ripgrep (`rg`)** (Parallel Regex File Search) — `en/`, `fa/` [MD + PDF]
- [ ] **jq** (High-Performance JSON Processor) — `en/`, `fa/` [MD + PDF]
- [ ] **git** (Plumbing & Porcelain Deep Diagnostics) — `en/`, `fa/` [MD + PDF]

---

## License

MIT © 2026 [Fernie2000](https://github.com/Fernie2000). All technical manuals and vector styles are published under open permissive licensing.
