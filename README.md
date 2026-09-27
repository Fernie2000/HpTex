# PhTex — Definitive Developer Tools & Systems Reference

> **High-density, mathematically rigorous, production-grade technical manuals and quick-reference manifests for vital developer utilities.**

[![Language: Multi](https://img.shields.io/badge/Language-English%20%7C%20فارسی-blue.svg)](#tool-index--فهرست-ابزارها)
[![License: MIT](https://img.shields.io/badge/License-MIT-emerald.svg)](LICENSE)
[![Format: Markdown & Modern PDF](https://img.shields.io/badge/Format-MD%20%2B%20PDF-purple.svg)](#tool-index--فهرست-ابزارها)
[![Theme: Dynamic Light / Dark](https://img.shields.io/badge/HTML%20Theme-Light%20%2F%20Dark%20Switcher-indigo.svg)](#interactive-html-features)

---

## Directory Architecture

```text
PhTex/
├── README.md                      # Global entrypoint, tool taxonomy, shortcuts, sources
├── .agents/skills/                # Standardized Agent Skills for autonomous tool synthesis
├── en/                            # English technical manuals and vectors
│   ├── scrcpy/
│   │   ├── scrcpy.md              # Low-latency display virtualization & UHID spec
│   │   ├── scrcpy.html            # Standalone dynamic light/dark HTML viewer
│   │   └── scrcpy.pdf             # Modern vector PDF dossier
│   ├── scrapy/
│   │   ├── scrapy.md              # Asynchronous crawling specification & settings.py
│   │   ├── scrapy.html            # Standalone dynamic light/dark HTML viewer
│   │   └── scrapy.pdf             # Modern vector PDF dossier
│   └── tmux/
│       ├── tmux.md                # Exhaustive Markdown reference & production config
│       ├── tmux.html              # Standalone dynamic light/dark HTML viewer
│       └── tmux.pdf               # Stylized modern standalone PDF dossier
└── fa/                            # راهنماهای فنی و تخصصی به زبان فارسی
    ├── scrcpy/
    │   ├── scrcpy.md              # مستندات تخصصی انتقال تصویر و کنترل دستگاه‌های اندروید
    │   ├── scrcpy.html            # سورس HTML دوحالته لایت/دارک با ساختار کامل RTL
    │   └── scrcpy.pdf             # نسخه پی‌دی‌اف مدرن وکتور زبان فارسی
    ├── scrapy/
    │   ├── scrapy.md              # مستندات جامع وب اسکرپینگ، معماری و تنظیمات
    │   ├── scrapy.html            # سورس HTML دوحالته لایت/دارک با ساختار کامل RTL
    │   └── scrapy.pdf             # نسخه پی‌دی‌اف مدرن وکتور زبان فارسی
    └── tmux/
        ├── tmux.md                # مستندات جامع، معماری و کانفیگ بهینه‌سازی شده
        ├── tmux.html              # سورس HTML دوحالته لایت/دارک با ساختار کامل RTL
        └── tmux.pdf               # نسخه سند مدرن و طراحی‌شده‌ی اختصاصی PDF
```

---

## Tool Index / فهرست ابزارها

| Tool | Category | Language | Markdown Dossier | Modern PDF Dossier | Core Capability |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`tmux`** | Terminal Multiplexing | **EN** (English) | [English Guide](en/tmux/tmux.md) | [English PDF](en/tmux/tmux.pdf) | Client-server PTY management, persistent sessions, decoupled multiplexing |
| **`tmux`** | ترمینال مالتی‌پلکسر | **FA** (فارسی) | [راهنمای فارسی](fa/tmux/tmux.md) | [پی‌دی‌اف فارسی](fa/tmux/tmux.pdf) | مدیریت کلاینت-سرور PTY، سشن‌های پایدار، تقسیم پنل‌ها و بافرها |
| **`scrcpy`**| Android Virtualization | **EN** (English) | [English Guide](en/scrcpy/scrcpy.md) | [English PDF](en/scrcpy/scrcpy.pdf) | Low-latency display mirroring, MediaCodec hardware encoding, UHID/AOA input |
| **`scrcpy`**| کنترل و تصویر اندروید | **FA** (فارسی) | [راهنمای فارسی](fa/scrcpy/scrcpy.md) | [پی‌دی‌اف فارسی](fa/scrcpy/scrcpy.pdf) | مجازی‌سازی بلادرنگ تصویر، انکود سخت‌افزاری MediaCodec، شبیه‌سازی سخت‌افزاری UHID |
| **`scrapy`**| Asynchronous Web Crawling | **EN** (English) | [English Guide](en/scrapy/scrapy.md) | [English PDF](en/scrapy/scrapy.pdf) | Twisted/asyncio reactive scraping engine, XPath/CSS parsel algebra, AutoThrottle |
| **`scrapy`**| وب اسکرپینگ ناهمگام | **FA** (فارسی) | [راهنمای فارسی](fa/scrapy/scrapy.md) | [پی‌دی‌اف فارسی](fa/scrapy/scrapy.pdf) | موتور واکشی غیرمسدودکننده رخدادمحور، پایپ‌لاین اقلام، الگوریتم AutoThrottle |

---

## Interactive HTML Features

All standalone `.html` dossiers across `en/` and `fa/` directories incorporate a **Dynamic Light / Dark Theme Switcher**:
* **Browser View:** Click the floating toggle button (`🌓`) to switch between Dark Slate and Clean Light themes. The preference is persisted in browser `localStorage`.
* **Print / Vector PDF Compilation:** Strictly locked to the high-contrast light theme under `@media print` to guarantee flawless vector rendering and zero toner/ink waste when rasterizing.

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

---

## scrcpy Quick Reference & Shortcuts

Default modifier key is `MOD` = `Alt` (or `Super` on macOS/Linux).

| Action | CLI Flag / Hotkey | Mechanical Effect |
| :--- | :--- | :--- |
| **Wireless Transport** | `scrcpy --tcpip=192.168.1.X:5555` | Disconnects USB; streams over ADB TCP/IP. |
| **Toggle Fullscreen** | <kbd>MOD</kbd> + <kbd>f</kbd> | Toggles borderless fullscreen SDL2 canvas. |
| **Android Navigation** | <kbd>MOD</kbd> + <kbd>h</kbd> / <kbd>b</kbd> / <kbd>s</kbd> | Injects Home, Back, or Overview (Recent Apps). |
| **Turn Screen Off** | `scrcpy -S` or <kbd>MOD</kbd> + <kbd>o</kbd> | Powers off physical panel backlight; preserves stream. |
| **Clipboard Sync** | <kbd>MOD</kbd> + <kbd>c</kbd> / <kbd>v</kbd> | Bidirectional host/device clipboard paste & copy. |
| **Direct APK Install** | Drag & Drop APK file | Directly executes streaming `adb install -r`. |
| **Webcam Emulation** | `scrcpy --video-source=camera --v4l2-sink` | Streams high-fidelity sensor feed as virtual webcam. |

---

## Scrapy CLI Quick Reference

| Command | Scope | Syntax Pattern | Target Action |
| :--- | :--- | :--- | :--- |
| **`crawl`** | Project | `scrapy crawl <spider> -O out.jsonl` | Boots reactor, parses target, streams items to file. |
| **`shell`** | Global | `scrapy shell "<url>" --nolog` | Interactive XPath/CSS evaluation in IPython. |
| **`fetch`** | Global | `scrapy fetch --headers "<url>"` | Low-level HTTP response inspection. |
| **`view`** | Global | `scrapy view "<url>"` | Verifies browser rendering against raw downloader response. |
| **`parse`** | Project | `scrapy parse --spider=<sp> <url>` | Direct contract and callback verification. |

---

## Curated Sources & Primary Specifications

* **tmux Official Repository:** [github.com/tmux/tmux](https://github.com/tmux/tmux)
* **scrcpy Official Repository:** [github.com/Genymobile/scrcpy](https://github.com/Genymobile/scrcpy)
* **Scrapy Official Repository:** [github.com/scrapy/scrapy](https://github.com/scrapy/scrapy)
* **OpenBSD Manual Pages:** [man.openbsd.org/tmux.1](https://man.openbsd.org/tmux.1)
* **Android Input Subsystem & UHID:** [source.android.com/devices/input](https://source.android.com/devices/input)
* **Twisted Asynchronous Network Engine:** [github.com/twisted/twisted](https://github.com/twisted/twisted)

---

## Roadmap

- [x] **tmux** (Terminal Multiplexer) — `en/`, `fa/` [MD + PDF]
- [x] **scrcpy** (Low-Latency Screen Copy) — `en/`, `fa/` [MD + PDF]
- [x] **scrapy** (Asynchronous Web Scraping) — `en/`, `fa/` [MD + PDF]
- [ ] **neovim** (Modal Hyperextensible Editor) — `en/`, `fa/` [MD + PDF]
- [ ] **fzf** (Command-line Fuzzy Finder) — `en/`, `fa/` [MD + PDF]
- [ ] **ripgrep (`rg`)** (Parallel Regex File Search) — `en/`, `fa/` [MD + PDF]
- [ ] **jq** (High-Performance JSON Processor) — `en/`, `fa/` [MD + PDF]
- [ ] **git** (Plumbing & Porcelain Deep Diagnostics) — `en/`, `fa/` [MD + PDF]

---

## License

MIT © 2026 [Fernie2000](https://github.com/Fernie2000). All technical manuals and vector styles are published under open permissive licensing.
