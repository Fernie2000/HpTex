# Technical Specification & Operational Manual: `tmux` (Terminal Multiplexer)

> **Document Class:** Engineering Reference Standard  
> **Domain:** UNIX PTY Multiplexing, Session Persistence, TTY State Virtualization  
> **Target Release:** `tmux >= 3.2`

---

## 1. Architectural Anatomy & Topology

`tmux` is not merely a window splitter; it is an event-driven terminal virtualization server operating atop POSIX pseudo-terminals (`pty`). It decouples interactive user shells from the physical or graphical terminal emulator lifecycle.

```
       +---------------------------------------------+
       |             tmux Server Process             |
       |  (Owns unix socket: /tmp/tmux-$UID/default) |
       +---------------------------------------------+
                              |
       +----------------------+----------------------+
       |                                             |
+--------------+                              +--------------+
| Session 'dev'|                              | Session 'ops'|
+--------------+                              +--------------+
       |                                             |
+--------------+                              +--------------+
| Window 1..N  |                              | Window 1..N  |
+--------------+                              +--------------+
       |                                             |
+--------------+                              +--------------+
| Pane 1..M    |                              | Pane 1..M    |
| (Bound pty)  |                              | (Bound pty)  |
+--------------+                              +--------------+
       ^                                             ^
       |                                             |
 [Client A (TTY)]                              [Client B (SSH)]
```

### Core Hierarchy & State Entities

1. **Server (`tmuxd`):** A standalone background daemon managing all client sockets, window layouts, buffers, and pty descriptors. Survived by active sessions even if all client connections terminate.
2. **Session:** A collection of one or more logical windows grouped under a shared identifier.
3. **Window:** A singular screen canvas corresponding to an application tab. Contains one or more panes partitioned geometrically.
4. **Pane:** A discrete pseudo-terminal running a dedicated shell or command process (e.g., `bash`, `zsh`, `nvim`).
5. **Client:** An interactive TTY process connected to the server via the UNIX domain socket (`/tmp/tmux-$(id -u)/default`).

---

## 2. Command Grammar & Control Matrix

Interactions occur through two distinct pathways:
* **Out-of-band CLI commands:** Invoked from external shells or automation scripts.
* **In-band Keybindings:** Triggered via the **Prefix** key (default: `Ctrl+b`, optimized: `Ctrl+a`), followed by a command chord.

### 2.1 Session Control

| Action | CLI Command | Interactive Chord | Execution Logic |
| :--- | :--- | :--- | :--- |
| **Initialize Session** | `tmux new-session -s <name>` | `:new -s <name>` | Spawns server (if dormant) and binds initial window. |
| **List Sessions** | `tmux ls` | `Prefix` + `s` | Queries server socket; renders interactive session tree. |
| **Attach Session** | `tmux attach -t <name>` | — | Binds current TTY client to existing target session. |
| **Detach Client** | `tmux detach` | `Prefix` + `d` | Unlinks TTY client; server and background processes persist. |
| **Kill Session** | `tmux kill-session -t <name>` | `:kill-session` | Sends SIGHUP/SIGKILL downstream to all managed ptys. |
| **Rename Session** | `tmux rename-session -t <old> <new>` | `Prefix` + `$` | Mutates target session identifier. |

### 2.2 Window Virtualization

| Action | CLI / Command Mode | Keybinding | Technical Nuance |
| :--- | :--- | :--- | :--- |
| **Create Window** | `tmux new-window -n <name>` | `Prefix` + `c` | Allocates new virtual screen index. |
| **Next / Previous** | `tmux next-window` / `tmux prev-window` | `Prefix` + `n` / `Prefix` + `p` | Circular index transversal. |
| **Direct Index Jump** | `tmux select-window -t <idx>` | `Prefix` + `<0-9>` | O(1) jump by window slot. |
| **Interactive Selection** | `tmux choose-tree -w` | `Prefix` + `w` | Hierarchical modal navigation menu. |
| **Rename Window** | `tmux rename-window <name>` | `Prefix` + `,` | Overrides automated process name deduction. |
| **Destroy Window** | `tmux kill-window` | `Prefix` + `&` | Closes all child panes simultaneously. |

### 2.3 Pane Algebra & Tiling

| Action | Default Chord | Optimized Chord (`.tmux.conf`) | Under-the-hood Command |
| :--- | :--- | :--- | :--- |
| **Vertical Split (Side-by-Side)** | `Prefix` + `%` | `Prefix` + `\|` | `split-window -h -c "#{pane_current_path}"` |
| **Horizontal Split (Stacked)** | `Prefix` + `"` | `Prefix` + `-` | `split-window -v -c "#{pane_current_path}"` |
| **Directional Focus** | `Prefix` + `Arrows` | `Prefix` + `h/j/k/l` | `select-pane -[U\|D\|L\|R]` |
| **Zoom Toggle** | `Prefix` + `z` | `Prefix` + `z` | `resize-pane -Z` (maximizes pane; retains geometry) |
| **Cycle Layout Presets** | `Prefix` + `Space` | `Prefix` + `Space` | `next-layout` (even-horiz, even-vert, main-horiz, etc.) |
| **Synchronize Inputs** | `:setw synchronize-panes` | `Prefix` + `y` | Broadcasts stdin across all panes in current window. |
| **Kill Pane** | `Prefix` + `x` | `Prefix` + `x` | Sends SIGHUP to the active pane's foreground process. |

---

## 3. The Vi Copy Mode & Clipboard Pipeline

`tmux` isolates scrollback history inside server memory, bypassing native terminal scroll limits.

### Configuration for Vi Operations
```tmux
set-window-option -g mode-keys vi
bind-key -T copy-mode-vi v send -X begin-selection
bind-key -T copy-mode-vi y send -X copy-pipe-and-cancel "xclip -in -selection clipboard"
```

### Execution Workflow

```
[ Normal Terminal Mode ]
          │  Prefix + [
          ▼
[ Copy / Scrollback Mode ]
          │  Navigate with h, j, k, l, Ctrl+u, Ctrl+d, /, ?
          │  Press 'v'
          ▼
[ Active Selection Range ]
          │  Press 'y'
          ▼
[ Extracted to Tmux Buffer & OS Clipboard via OSC-52 / xclip ]
          │  Prefix + ]
          ▼
[ Injected into Foreground TTY ]
```

---

## 4. Hardened, Production-Grade `.tmux.conf`

Deploy this configuration to `~/.tmux.conf` for deterministic, low-latency, 24-bit truecolor environments.

```tmux
# ==============================================================================
# PhTex High-Performance Production .tmux.conf
# ==============================================================================

# 1. Prefix Rebinding (Ergonomic Ctrl-a mapping)
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# 2. Terminal Capabilities & Truecolor (24-bit RGB)
set -g default-terminal "tmux-256color"
set -ag terminal-overrides ",xterm-256color:RGB"

# 3. Latency Optimization (Eliminate Esc delay for Neovim/Vim)
set -s escape-time 0
set -g repeat-time 300

# 4. Ergonomic Window & Pane Indexing (1-based instead of 0-based)
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on

# 5. Increased History Buffer Size
set -g history-limit 50000

# 6. Mouse Integration (Scroll, Resize, Select)
set -g mouse on

# 7. Intuitive Split Commands (Retain current working directory)
unbind '"'
unbind %
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# 8. Vim-Style Pane Navigation
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# 9. Pane Resizing with Vi Chords
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# 10. Synchronize Panes Toggle
bind y setw synchronize-panes

# 11. Vi Mode for Buffers & OSC 52 System Clipboard Pass-through
setw -g mode-keys vi
bind -T copy-mode-vi v send-keys -X begin-selection
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "wl-copy || xclip -in -selection clipboard"
set -s set-clipboard on

# 12. Minimalist, High-Contrast Status Bar Design
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

# 13. Fast Config Reload Shortcut
bind r source-file ~/.tmux.conf \; display-message "Configuration Reloaded Successfully."
```

---

## 5. Non-Interactive Automation & Scripting

Because `tmux` commands are first-class CLI citizens, complex development workspaces can be deterministically provisioned through shell routines.

### Automated Workspace Bootstrap (`dev-workspace.sh`)

```bash
#!/usr/bin/env bash
SESSION_NAME="infrastructure"

# Prevent duplicate spawning
tmux has-session -t "$SESSION_NAME" 2>/dev/null && {
    echo "Attaching to existing session '$SESSION_NAME'..."
    exec tmux attach-session -t "$SESSION_NAME"
}

# 1. Initialize detached master session with editor window
tmux new-session -d -s "$SESSION_NAME" -n "editor" -c "$HOME/projects/core"

# 2. Window 2: Diagnostics & Services (Split 65/35)
tmux new-window -n "services" -t "$SESSION_NAME:2" -c "$HOME/projects/core"
tmux split-window -h -t "$SESSION_NAME:2" -p 35 -c "$HOME/projects/core"
tmux send-keys -t "$SESSION_NAME:2.1" "htop" C-m
tmux send-keys -t "$SESSION_NAME:2.2" "tail -f /var/log/syslog" C-m

# 3. Window 3: Git Porcelain & Shell
tmux new-window -n "vcs" -t "$SESSION_NAME:3" -c "$HOME/projects/core"

# 4. Focus primary window and bind interactive terminal
tmux select-window -t "$SESSION_NAME:1"
exec tmux attach-session -t "$SESSION_NAME"
```

---

## 6. Diagnostic Matrix & Common Pitfalls

| Symptom | Root Cause | Definitive Remediation |
| :--- | :--- | :--- |
| **`ESC` key delay in Vim/Neovim** | `tmux` waits for escape sequences to differentiate meta-keys. | Set `set -s escape-time 0` in `~/.tmux.conf`. |
| **Color corruption / 256 colors missing** | `$TERM` variable mismatch or outer terminal lacking RGB caps. | Enforce `set -g default-terminal "tmux-256color"` and pass `terminal-overrides`. |
| **Nested Session Key Hijacking** | Inner `tmux` receives outer prefix keystroke. | Press `Prefix` twice (`Ctrl-a Ctrl-a`) to pass prefix downward to the guest session. |
| **Window dimensions constrained by smaller client** | `tmux` defaults to sizing windows to smallest attached client viewport. | Enable aggressive resizing: `setw -g aggressive-resize on`. |
