# مشخصات فنی و راهنمای جامع عملیاتی: `tmux` (Terminal Multiplexer)

> **رده سند:** مرجع استاندارد مهندسی سیستم  
> **حوزه تخصصی:** مالتی‌پلکسینگ پایانه‌های یونیکس (PTY)، پایداری نشست‌ها، مجازی‌سازی کلاینت/سرور TTY  
> **نسخه هدف:** `tmux >= 3.2`

---

## ۱. آناتومی معماری و توپولوژی سیستم

ابزار `tmux` صرفاً یک نرم‌افزار برای تقسیم صفحه نمایش (Window Splitting) نیست؛ بلکه یک سرور مستقل مدیریت ترمینال مبتنی بر رخداد (Event-driven) است که روی شبه‌پایانه‌های استاندارد پازیکس (`pty`) قرار می‌گیرد. این معماری، فرآیند اجرای شل و دستورات را از طول عمر پنجره فیزیکی یا گرافیکی ترمینال شبیه‌ساز جدا می‌کند.

```text
       +---------------------------------------------+
       |             tmux Server Process             |
       |  (سوکت یونیکس: /tmp/tmux-$UID/default)       |
       +---------------------------------------------+
                              |
       +----------------------+----------------------+
       |                                             |
+--------------+                              +--------------+
| نشست 'dev'   |                              | نشست 'ops'   |
+--------------+                              +--------------+
       |                                             |
+--------------+                              +--------------+
| پنجره ۱..N   |                              | پنجره ۱..N   |
+--------------+                              +--------------+
       |                                             |
+--------------+                              +--------------+
| پنل ۱..M     |                              | پنل ۱..M     |
| (متصل به pty)|                              | (متصل به pty)|
+--------------+                              +--------------+
       ^                                             ^
       |                                             |
 [کلاینت A (TTY)]                              [کلاینت B (SSH)]
```

### ساختار و موجودیت‌های اصلی

۱. **سرور (`tmuxd`):** فرآیندی که در پس‌زمینه (Background Daemon) اجرا شده و سوکت‌های یونیکس، پنجره‌ها، حافظه بافر و توصیف‌کننده‌های PTY را مدیریت می‌کند. حتی اگر ارتباط تمام کلاینت‌ها قطع شود، پردازه‌های سرور بدون وقفه زنده می‌مانند.  
۲. **نشست (Session):** مجموعه‌ای از پنجره‌های منطقی که تحت یک شناسه واحد دسته‌بندی شده‌اند.  
۳. **پنجره (Window):** صفحه‌ای مستقل مشابه تب‌های مرورگر که شامل یک یا چند پنل هندسی است.  
۴. **پنل (Pane):** یک شبیه‌ساز پایانه مجزا که یک دستور یا پوسته اختصاصی (مانند `bash`، `zsh` یا `nvim`) را میزبانی می‌کند.  
۵. **کلاینت (Client):** فرآیند تعاملی ترمینال کاربر که از طریق یونیکس دامین سوکت (`/tmp/tmux-$(id -u)/default`) به سرور متصل می‌شود.

---

## ۲. گرامر دستورات و جدول کلیدهای میانبر

تعامل با `tmux` از دو مجرای مجزا صورت می‌پذیرد:
* **دستورات خط فرمان (CLI):** قابل اجرا از محیط شل یا اسکریپت‌های اتوماسیون.
* **کلیدهای میانبر درون‌برنامه‌ای:** با فشردن کلید **پیشوند (Prefix)** (به‌صورت پیش‌فرض `Ctrl+b` و در کانفیگ بهینه‌سازی‌شده `Ctrl+a`) و سپس کلید دستوری فعال می‌شوند.

### ۲.۱ مدیریت چرخه حیات نشست‌ها (Sessions)

| عملیات | دستور CLI | کلید تعاملی | منطق و نحوه اجرا |
| :--- | :--- | :--- | :--- |
| **ایجاد نشست جدید با نام مشخص** | `tmux new-session -s <name>` | `:new -s <name>` | راه‌اندازی سرور (در صورت غیرفعال بودن) و ساخت نخستین پنجره. |
| **مشاهده فهرست نشست‌ها** | `tmux ls` | `Prefix` + `s` | کوئری از سوکت سرور و نمایش درختی نشست‌ها. |
| **اتصال به نشست موجود** | `tmux attach -t <name>` | — | اتصال کلاینت ترمینال فعال به نشست هدف. |
| **جدا شدن از نشست (Detach)** | `tmux detach` | `Prefix` + `d` | قطع ارتباط کلاینت بدون بستن پروسه‌ها و ادامه کار در پس‌زمینه. |
| **پایان دادن به نشست** | `tmux kill-session -t <name>` | `:kill-session` | ارسال سیگنال `SIGHUP`/`SIGKILL` به کلیه PTYهای تابعه. |
| **تغییر نام نشست** | `tmux rename-session -t <old> <new>` | `Prefix` + `$` | ویرایش شناسه نامی نشست فعال یا مشخص‌شده. |

### ۲.۲ مدیریت پنجره‌ها (Windows)

| عملیات | دستور CLI / حالت Command Mode | کلید میانبر | عملکرد سیستمی |
| :--- | :--- | :--- | :--- |
| **ساخت پنجره جدید** | `tmux new-window -n <name>` | `Prefix` + `c` | تخصیص یک اسلات ایندکس و بوم نمایش جدید. |
| **پنجره بعدی / قبلی** | `tmux next-window` / `tmux prev-window` | `Prefix` + `n` / `Prefix` + `p` | جابجایی چرخشی میان پنجره‌های فعال. |
| **پرش به ایندکس عددی** | `tmux select-window -t <idx>` | `Prefix` + `<0-9>` | سوئیچ بدون تاخیر (O(1)) به اسلات شماره‌گذاری‌شده. |
| **منوی انتخاب سلسله‌مراتبی** | `tmux choose-tree -w` | `Prefix` + `w` | باز کردن منوی درخت ناوبری پنجره‌ها. |
| **تغییر نام پنجره جاری** | `tmux rename-window <name>` | `Prefix` + `,` | لغو استنتاج خودکار نام بر اساس فرآیند foreground. |
| **حذف و بستن پنجره** | `tmux kill-window` | `Prefix` + `&` | نابودی هم‌زمان تمام پنل‌های درون پنجره. |

### ۲.۳ هندسه و تقسیم‌بندی پنل‌ها (Panes)

| عملیات | کلید پیش‌فرض | کلید بهینه‌شده (`.tmux.conf`) | دستور معادل زیرین |
| :--- | :--- | :--- | :--- |
| **تقسیم افقی (کنار هم / عمودی ظاهری)** | `Prefix` + `%` | `Prefix` + `\|` | `split-window -h -c "#{pane_current_path}"` |
| **تقسیم عمودی (زیر هم / افقی ظاهری)** | `Prefix` + `"` | `Prefix` + `-` | `split-window -v -c "#{pane_current_path}"` |
| **جابجایی جهت‌دار فوکوس** | `Prefix` + `جهت‌ها` | `Prefix` + `h/j/k/l` | `select-pane -[U\|D\|L\|R]` |
| **حالت بزرگ‌نمایی (Toggle Zoom)** | `Prefix` + `z` | `Prefix` + `z` | `resize-pane -Z` (تمام‌صفحه کردن پنل بدون تغییر چیدمان) |
| **تغییر چیدمان‌های پیش‌فرض (Layout)** | `Prefix` + `Space` | `Prefix` + `Space` | `next-layout` (حالت‌های افقی، عمودی، پنجره اصلی و ...) |
| **همگام‌سازی ورودی پنل‌ها** | `:setw synchronize-panes` | `Prefix` + `y` | ارسال همزمان ورودی کیبورد به تمامی پنل‌های پنجره. |
| **بستن پنل فعال** | `Prefix` + `x` | `Prefix` + `x` | ارسال سیگنال `SIGHUP` به پردازه در حال اجرا در پنل. |

---

## ۳. مدیریت حافظه بافر و حالت کپی Vi

ابزار `tmux` تاریخچه متنی خروجی‌ها (Scrollback) را در حافظه سرور ایزوله می‌کند؛ بنابراین بدون وابستگی به بافر شبیه‌ساز ترمینال، امکان پیمایش نامحدود فراهم است.

### تنظیمات رفتار Vi در کانفیگ
```tmux
set-window-option -g mode-keys vi
bind-key -T copy-mode-vi v send -X begin-selection
bind-key -T copy-mode-vi y send -X copy-pipe-and-cancel "xclip -in -selection clipboard"
```

### فلوچارت عملیاتی انتخاب و کپی

```text
[ حالت ترمینال معمولی ]
          │  فشردن Prefix + [
          ▼
[ ورود به حالت Copy / Scrollback ]
          │  ناوبری با کلیدهای h, j, k, l, Ctrl+u, Ctrl+d یا جستجو با /
          │  فشردن کلید 'v'
          ▼
[ شروع انتخاب متن (Selection) ]
          │  فشردن کلید 'y'
          ▼
[ ذخیره در بافر داخلی tmux و کلیپ‌بورد سیستم عامل با xclip / wl-copy ]
          │  فشردن Prefix + ]
          ▼
[ الصاق (Paste) در محیط ترمینال فعال ]
```

---

## ۴. فایل پیکربندی پایدار و آماده برای محیط تولید (`.tmux.conf`)

این پیکربندی استانداردهای نوین (Truecolor 24-bit، پیشوند ارگونومیک، کاهش تاخیر کلید Escape به صفر و شماره‌گذاری استاندارد از ۱) را اعمال می‌کند. این محتوا را در مسیر `~/.tmux.conf` ذخیره کنید:

```tmux
# ==============================================================================
# PhTex High-Performance Production .tmux.conf
# ==============================================================================

# ۱. تغییر پیشوند به کلید ارگونومیک Ctrl-a
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# ۲. پشتیبانی کامل از رنگ‌های ۲۴-بیتی مدرن (Truecolor RGB)
set -g default-terminal "tmux-256color"
set -ag terminal-overrides ",xterm-256color:RGB"

# ۳. حذف کامل تاخیر کلید Esc برای ویرایشگرهای Vim و Neovim
set -s escape-time 0
set -g repeat-time 300

# ۴. شماره‌گذاری پنجره‌ها و پنل‌ها از ۱ به جای ۰ (مطابق ردیف کیبورد)
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on

# ۵. افزایش ظرفیت تاریخچه اسکرول به ۵۰,۰۰۰ سطر
set -g history-limit 50000

# ۶. فعال‌سازی پشتیبانی از ماوس (اسکرول، تغییر ابعاد، انتخاب)
set -g mouse on

# ۷. تقسیم پنجره با کلیدهای بصری و حفظ دایرکتوری جاری
unbind '"'
unbind %
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# ۸. ناوبری سریع بین پنل‌ها به شیوه ادیتور Vim
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# ۹. تغییر ابعاد پنل‌ها با نگه داشتن کلیدهای پیشوند و حروف بزرگ
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# ۱۰. سوئیچ همگام‌سازی ورودی تمام پنل‌ها
bind y setw synchronize-panes

# ۱۱. فعال‌سازی حالت Vi برای کپی و اتصال به کلیپ‌بورد سیستم
setw -g mode-keys vi
bind -T copy-mode-vi v send-keys -X begin-selection
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "wl-copy || xclip -in -selection clipboard"
set -s set-clipboard on

# ۱۲. استایل نوار وضعیت مدرن و مینیمال (Catppuccin Mocha Theme)
set -g status-position bottom
set -g status-justify left
set -g status-interval 3
set -g status-style "bg=#1e1e2e,fg=#cdd6f4"

set -g status-left "#[bg=#89b4fa,fg=#11111b,bold]  #S #[bg=#1e1e2e,fg=#89b4fa]"
set -g status-left-length 30

set -g window-status-format "#[fg=#6c7086,bg=#1e1e2e] #I:#W "
set -g window-status-current-format "#[fg=#11111b,bg=#a6e3a1,bold] #I:#W #[fg=#a6e3a1,bg=#1e1e2e]"

set -g status-right "#[fg=#f38ba8,bg=#1e1e2e] %Y-%m-%d #[fg=#cdd6f4]| #[fg=#fab387]%H:%M:%S #[bg=#b4befe,fg=#11111b,bold] #H "
set -g status-right-length 60

# ۱۳. بارگذاری مجدد و آنی کانفیگ با فشردن r
bind r source-file ~/.tmux.conf \; display-message "کانفیگ با موفقیت مجدداً بارگذاری شد."
```

---

## ۵. اتوماسیون و اسکریپت‌نویسی محیط کار

به دلیل ساختار CLI ابزار `tmux`، می‌توان محیط‌های پیچیده کاری شامل سرورها، لاگ‌ها و ویرایشگرها را به شکل کاملاً خودکار در قالب یک فایل اسکریپت `bash` بالا آورد.

### نمونه اسکریپت خودکارسازی محیط توسعه (`dev-workspace.sh`)

```bash
#!/usr/bin/env bash
SESSION_NAME="infrastructure"

# بررسی عدم وجود سشن از قبل جهت جلوگیری از اجرای تکراری
tmux has-session -t "$SESSION_NAME" 2>/dev/null && {
    echo "سشن '$SESSION_NAME' از قبل فعال است؛ در حال اتصال..."
    exec tmux attach-session -t "$SESSION_NAME"
}

# ۱. ساخت نشست در پس‌زمینه با پنجره اول برای ویرایشگر کد
tmux new-session -d -s "$SESSION_NAME" -n "editor" -c "$HOME/projects/core"

# ۲. ساخت پنجره دوم برای سرویس‌ها و مانیتورینگ سیستم (تقسیم ۳۵/۶۵)
tmux new-window -n "services" -t "$SESSION_NAME:2" -c "$HOME/projects/core"
tmux split-window -h -t "$SESSION_NAME:2" -p 35 -c "$HOME/projects/core"
tmux send-keys -t "$SESSION_NAME:2.1" "htop" C-m
tmux send-keys -t "$SESSION_NAME:2.2" "tail -f /var/log/syslog" C-m

# ۳. ساخت پنجره سوم برای مدیریت نسخه و گیت
tmux new-window -n "vcs" -t "$SESSION_NAME:3" -c "$HOME/projects/core"

# ۴. فوکوس روی پنجره اول و اتصال نهایی کلاینت
tmux select-window -t "$SESSION_NAME:1"
exec tmux attach-session -t "$SESSION_NAME"
```

---

## ۶. جدول عیب‌یابی و پیشگیری از خطاهای رایج

| نشانه و خطا | ریشه فنی مشکل | راه‌حل قطعی |
| :--- | :--- | :--- |
| **تاخیر کلید `ESC` در Vim/Neovim** | `tmux` برای شناسایی کلیدهای ترکیبی منتظر کاراکتر بعدی می‌ماند. | مقدار `set -s escape-time 0` را در `~/.tmux.conf` قرار دهید. |
| **ناهمگونی رنگ‌ها یا عدم نمایش ۲۴-بیتی** | عدم تطابق مقدار متغیر محیطی `$TERM` با ترمینال اصلی میزبان. | اعمال `set -g default-terminal "tmux-256color"` همراه با تنظیمات RGB overrides. |
| **تداخل کلید پیشوند در نشست‌های تودرتو (SSH)** | پیشوند توسط `tmux` والد یا سیستم میزبان ضبط می‌شود. | فشردن دو بار متوالی کلید پیشوند (`Ctrl-a Ctrl-a`) سیگنال را به سرور دوم منتقل می‌کند. |
| **محدود شدن ابعاد پنجره به کوچکترین کلاینت** | `tmux` به طور پیش‌فرض اندازه پنجره را به کوچکترین کلاینت متصل محدود می‌کند. | فعال‌سازی تغییر ابعاد شناور با دستور `setw -g aggressive-resize on`. |
