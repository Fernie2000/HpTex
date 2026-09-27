# HpTex — جعبه‌ابزار و چیت‌شیت‌های سریع برنامه‌نویسی

> **دیگه لازم نیست برای شورت‌کات‌های tmux، تنظیمات scrcpy یا کدهای اسکرپی ۲۰ تا تب گوگل باز کنی!  
> هپتکس (HpTex) مجموعه‌ای از راهنماهای تصویری و ۲ صفحه‌ای (PDF و وب تعاملی) برای ابزارهای پرکاربرد توسعه‌دهنده‌هاست.**

[![Language: Multi](https://img.shields.io/badge/Language-English%20%7C%20فارسی-blue.svg)](#tool-index--فهرست-ابزارها)
[![License: MIT](https://img.shields.io/badge/License-MIT-emerald.svg)](LICENSE)
[![Format: PDF & Web](https://img.shields.io/badge/Format-PDF%20%2B%20Web-purple.svg)](#tool-index--فهرست-ابزارها)
[![Theme: Light / Dark](https://img.shields.io/badge/Theme-Light%20%2F%20Dark-indigo.svg)](#interactive-html-features)

---

### 💡 هپتکس (HpTex) دقیقاً چیه؟ / What is HpTex?

فرض کن وسط کدنویسی یا کانفیگ سرور هستی:
* کلیدهای تقسیم صفحه در **tmux** یادت رفته؟
* می‌خوای صفحه گوشی رو با **scrcpy** بندازی رو مانیتور ولی دستور اتصال بی‌سیم یا وب‌کمش یادت نیست؟
* می‌خوای با **Scrapy** داده کراول کنی و نمی‌خوای رم سرور منفجر بشه یا آی‌پیت بلاک بشه؟

**هپتکس دقیقاً برای همین ساخته شده:**  
به جای گشتن تو سایت‌های مختلف و خوندن متن‌های طولانی، برای هر ابزار یک بسته شسته و رفته داری:
1. **یک فایل PDF دو صفحه‌ای شیک و تمیز:** خلاصه تمام دستورات، کلیدها و رفع خطاهای رایج (آماده ذخیره روی دسکتاپ یا پرینت).
2. **یک صفحه وب HTML مدرن:** با امکان تغییر سریع حالت دارک / لایت (Dark/Light).
3. **راهنمای متنی کامل (Markdown):** به دو زبان **فارسی روان** و **انگلیسی** به همراه کانفیگ‌های تست‌شده و آماده کپی‌پیست.


## Directory Architecture: `HpTex/<Tool>/<Language>/<Files>`

```text
HpTex/
├── README.md                      # Global entrypoint, tool taxonomy, shortcuts, sources
├── .agents/skills/                # Standardized Agent Skills for autonomous tool synthesis
├── scrcpy/                        # Low-Latency Android Screen Copy & HID Subsystem
│   ├── en/
│   │   ├── scrcpy.md              # Low-latency display virtualization & UHID spec
│   │   ├── scrcpy.html            # Standalone dynamic light/dark HTML viewer
│   │   └── scrcpy.pdf             # Modern vector PDF dossier
│   └── fa/
│       ├── scrcpy.md              # مستندات تخصصی انتقال تصویر و کنترل دستگاه‌های اندروید
│       ├── scrcpy.html            # سورس HTML دوحالته لایت/دارک با ساختار کامل RTL
│       └── scrcpy.pdf             # نسخه پی‌دی‌اف مدرن وکتور زبان فارسی
├── scrapy/                        # Asynchronous Web Crawling & Scraping Framework
│   ├── en/
│   │   ├── scrapy.md              # Asynchronous crawling specification & settings.py
│   │   ├── scrapy.html            # Standalone dynamic light/dark HTML viewer
│   │   └── scrapy.pdf             # Modern vector PDF dossier
│   └── fa/
│       ├── scrapy.md              # مستندات جامع وب اسکرپینگ، معماری و تنظیمات
│       ├── scrapy.html            # سورس HTML دوحالته لایت/دارک با ساختار کامل RTL
│       └── scrapy.pdf             # نسخه پی‌دی‌اف مدرن وکتور زبان فارسی
└── tmux/                          # Terminal Multiplexing Engine
    ├── en/
    │   ├── tmux.md                # Exhaustive Markdown reference & production config
    │   ├── tmux.html              # Standalone dynamic light/dark HTML viewer
    │   └── tmux.pdf               # Stylized modern standalone PDF dossier
    └── fa/
        ├── tmux.md                # مستندات جامع، معماری و کانفیگ بهینه‌سازی شده
        ├── tmux.html              # سورس HTML دوحالته لایت/دارک با ساختار کامل RTL
        └── tmux.pdf               # نسخه سند مدرن و طراحی‌شده‌ی اختصاصی PDF
```

---

## Tool Index / فهرست ابزارها

| Tool | Category | Language | Markdown Dossier | Modern PDF Dossier | Core Capability |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`tmux`** | Terminal Multiplexing | **EN** (English) | [English Guide](tmux/en/tmux.md) | [English PDF](tmux/en/tmux.pdf) | Client-server PTY management, persistent sessions, decoupled multiplexing |
| **`tmux`** | ترمینال مالتی‌پلکسر | **FA** (فارسی) | [راهنمای فارسی](tmux/fa/tmux.md) | [پی‌دی‌اف فارسی](tmux/fa/tmux.pdf) | آموزش مو به موی تقسیم صفحه سیاه، کلیدهای کیبورد و زنده‌ماندن برنامه‌ها |
| **`scrcpy`**| Android Virtualization | **EN** (English) | [English Guide](scrcpy/en/scrcpy.md) | [English PDF](scrcpy/en/scrcpy.pdf) | Low-latency display mirroring, MediaCodec hardware encoding, UHID/AOA input |
| **`scrcpy`**| کنترل و تصویر اندروید | **FA** (فارسی) | [راهنمای فارسی](scrcpy/fa/scrcpy.md) | [پی‌دی‌اف فارسی](scrcpy/fa/scrcpy.pdf) | آموزش مو به مو از اتصال کابل تا تنظیمات مخفی گوشی و کنترل با ماوس/کیبورد |
| **`scrapy`**| Asynchronous Web Crawling | **EN** (English) | [English Guide](scrapy/en/scrapy.md) | [English PDF](scrapy/en/scrapy.pdf) | Twisted/asyncio reactive scraping engine, XPath/CSS parsel algebra, AutoThrottle |
| **`scrapy`**| وب اسکرپینگ و استخراج داده | **FA** (فارسی) | [راهنمای فارسی](scrapy/fa/scrapy.md) | [پی‌دی‌اف فارسی](scrapy/fa/scrapy.pdf) | آموزش ساخت ربات جمع‌آوری اطلاعات سایت‌ها و تبدیل مستقیم به فایل اکسل |

---

## Interactive HTML Features

All standalone `.html` dossiers across tools and languages incorporate a **Dynamic Light / Dark Theme Switcher**:
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

- [x] **tmux** (Terminal Multiplexer) — `tmux/{en,fa}/` [MD + PDF]
- [x] **scrcpy** (Low-Latency Screen Copy) — `scrcpy/{en,fa}/` [MD + PDF]
- [x] **scrapy** (Asynchronous Web Scraping) — `scrapy/{en,fa}/` [MD + PDF]
- [ ] **neovim** (Modal Hyperextensible Editor) — `neovim/{en,fa}/` [MD + PDF]
- [ ] **fzf** (Command-line Fuzzy Finder) — `fzf/{en,fa}/` [MD + PDF]
- [ ] **ripgrep (`rg`)** (Parallel Regex File Search) — `ripgrep/{en,fa}/` [MD + PDF]
- [ ] **jq** (High-Performance JSON Processor) — `jq/{en,fa}/` [MD + PDF]
- [ ] **git** (Plumbing & Porcelain Deep Diagnostics) — `git/{en,fa}/` [MD + PDF]

---

## License

MIT © 2026 [Fernie2000](https://github.com/Fernie2000). All technical manuals and vector styles are published under open permissive licensing.
