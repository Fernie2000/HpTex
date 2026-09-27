# Technical Specification & Operational Manual: `scrcpy` (Screen Copy)

> **Document Class:** Engineering Reference Standard  
> **Domain:** Low-Latency Display Virtualization, Android Subsystem Interop, ADB Tunneling  
> **Target Release:** `scrcpy >= 2.4` / `Android >= 5.0 (API 21)` (Audio: `Android >= 11 (API 30)`)

---

## 1. Architectural Anatomy & Subsystem Pipeline

`scrcpy` delivers real-time Android display mirroring and HID input control without requiring root access or companion apps installed on the device. It operates via an ephemeral in-memory Java server injected dynamically through the Android Debug Bridge (`adb`).

```text
+-------------------------------------------------------------------------+
|                              Host Machine                               |
|                                                                         |
|  [ SDL2 Renderer ] <=== Raw Frames === [ FFmpeg Hardware Decoder ]      |
|         ▲                                        ▲                      |
|         |                                        |                      |
|  [ User Input Events ]                 [ Demuxer / Parser ]             |
|         |                                        ▲                      |
|         ▼                                        |                      |
|  [ Scrapy Controller ] ── Control Packets ──> [ ADB Tunnel Socket ]    |
+------------------------------------------------------│------------------+
                                                       │  USB / TCP (5555)
+------------------------------------------------------│------------------+
|                            Android Device            ▼                  |
|                                             [ ADB Daemon (adbd) ]       |
|                                                      │                  |
|  +───────────────────────────────────────────────────┼───────────────+  |
|  │ scrcpy-server.jar (Executed via app_process /data/local/tmp)       │  |
|  │                                                                   │  |
|  │  [ SurfaceControl / DisplayManager ]                              │  |
|  │                     │ (Raw Surface)                               │  |
|  │                     ▼                                             │  |
|  │  [ MediaCodec Hardware Encoder ] ──> H.264 / H.265 / AV1 Stream   │  |
|  │                                                                   │  |
|  │  [ AudioRecord (AudioPlaybackCapture) ] ──> Opus / AAC Stream     │  |
|  │                                                                   │  |
|  │  [ InputManager / UHID / AOA Driver ] <── Input Event Injection   │  |
|  +───────────────────────────────────────────────────────────────────+  |
+-------------------------------------------------------------------------+
```

### Discrete Subsystem Mechanics

1. **Bootstrap Sequence:** The host pushes `scrcpy-server.jar` to `/data/local/tmp/scrcpy-server.jar` via `adb push`, then spawns it with `app_process` under shell credentials (`UID 2000`).
2. **Video Capture & Encoding:** Directly taps Android's private `SurfaceControl` or virtual display API, feeding raw surface buffers into the SoC's hardware `MediaCodec` encoder.
3. **Transport Layer:** Transmits raw NAL units (H.264, H.265, or AV1) and compressed audio (Opus, AAC, RAW) through an encrypted ADB socket forward (`adb forward` or `adb reverse`).
4. **Decoding & Display:** The host client utilizes FFmpeg hardware-accelerated decoders (DXVA2, NVDEC, VAAPI, VideoToolbox) and renders directly to an SDL2 GPU texture, achieving **35–70 ms** glass-to-glass latency.
5. **Input Injection Protocols:**
   * **`sdk` (Default):** Injects `MotionEvent` and `KeyEvent` through Android's internal `InputManager`.
   * **`uhid` / `aoa`:** Simulates native Linux kernel HID devices or Android Open Accessory hardware USB peripherals (bypasses software IME restrictions and keycode mapping flaws).

---

## 2. Command Grammar & Parameter Matrix

All operations interface via the `scrcpy` CLI binary. Arguments dictate transport topology, video compression constraints, and peripheral emulation.

| Flag / Parameter | Default | Operational Specification |
| :--- | :--- | :--- |
| **`-s, --serial=<serial>`** | First detected | Targets a specific device from `adb devices`. |
| **`--tcpip[=<ip:port>]`** | USB auto | Disconnects USB dependency; switches target to wireless TCP/IP mode. |
| **`-m, --max-size=<val>`** | `0` (Native) | Downscales display to fit bounding box (e.g. `1080`, `1440`), preserving aspect ratio. |
| **`-b, --video-bit-rate=<val>`** | `8M` | Sets H.264/H.265 video bitrate (e.g. `4M`, `16M`). |
| **`--max-fps=<val>`** | `0` (Uncapped) | Hard ceiling on frame production (e.g. `60`, `30`) to conserve SoC thermals and bandwidth. |
| **`--video-codec=<codec>`** | `h264` | Selects encoder: `h264`, `h265`, or `av1` (SoC hardware encoder dependent). |
| **`--no-audio`** | `false` | Disables audio streaming (mandatory for `Android < 11`). |
| **`-S, --turn-screen-off`** | `false` | Powers down physical device panel while mirroring remains active. |
| **`-w, --stay-awake`** | `false` | Prevents device sleeping via Wakelock while connected. |
| **`--record=<file.mp4>`** | None | Directly dumps bitstream to MP4 or MKV without client-side re-encoding. |
| **`--keyboard=<mode>`** | `sdk` | Input mode: `sdk`, `uhid`, or `aoa` (simulates physical USB hardware keyboard). |
| **`--video-source=camera`** | `display` | Uses device camera sensors as a high-fidelity USB webcam. |

---

## 3. Interactive Modal Keybindings

Interactive controls rely on the **Modifier Key (`MOD`)**, which defaults to `Alt` (or `Super` on macOS/Linux).

| Hotkey Combination | Action Trigger | Mechanical Effect |
| :--- | :--- | :--- |
| <kbd>MOD</kbd> + <kbd>f</kbd> | **Toggle Fullscreen** | Switches SDL canvas between windowed and borderless fullscreen. |
| <kbd>MOD</kbd> + <kbd>h</kbd> | **Trigger Home** | Injects `KEYCODE_HOME` into the foreground window manager. |
| <kbd>MOD</kbd> + <kbd>b</kbd> | **Trigger Back** | Injects `KEYCODE_BACK` (Right-click defaults to this behavior). |
| <kbd>MOD</kbd> + <kbd>s</kbd> | **App Switcher** | Opens Android Multitasking / Recent Apps view. |
| <kbd>MOD</kbd> + <kbd>p</kbd> | **Power Toggle** | Emulates hardware Power button press (`KEYCODE_POWER`). |
| <kbd>MOD</kbd> + <kbd>o</kbd> | **Blank Physical Screen** | Turns physical panel backlight off; continues stream rendering. |
| <kbd>MOD</kbd> + <kbd>Up</kbd> / <kbd>Down</kbd> | **Volume Increment** | Adjusts target hardware audio attenuation. |
| <kbd>MOD</kbd> + <kbd>n</kbd> / <kbd>Shift+n</kbd> | **Notification Tray** | Expands or collapses notification shade / quick settings. |
| <kbd>MOD</kbd> + <kbd>c</kbd> | **Copy Clipboard** | Synchronizes device clipboard contents to host operating system. |
| <kbd>MOD</kbd> + <kbd>v</kbd> | **Paste Clipboard** | Injects host clipboard contents directly into active Android text node. |
| <kbd>MOD</kbd> + <kbd>Shift+v</kbd> | **Inject as Keystrokes** | Converts clipboard UTF-8 payload to simulated hardware key events. |
| **Drag & Drop APK** | **Direct Package Install** | Executes streaming `adb install -r <file.apk>`. |
| **Drag & Drop File** | **Filesystem Push** | Pushes arbitrary files to `/sdcard/Download/`. |

---

## 4. Hardened Production Profiles & Automation Scripts

Deploy these deterministic invocation profiles to maximize bandwidth efficiency, eliminate latency, or record non-destructively.

### Low-Latency Wireless Control Profile (`scrcpy-wireless.sh`)

```bash
#!/usr/bin/env bash
# ==============================================================================
# PhTex Low-Latency Ultra-Responsive scrcpy Profile
# ==============================================================================
DEVICE_IP="${1:-192.168.1.150}"

echo "[*] Initializing wireless transport over TCP/IP..."
adb connect "${DEVICE_IP}:5555" || { echo "[!] ADB connect failed"; exit 1; }

exec scrcpy \
  --tcpip="${DEVICE_IP}:5555" \
  --video-codec=h265 \
  --max-size=1600 \
  --video-bit-rate=6M \
  --max-fps=60 \
  --audio-codec=opus \
  --audio-bit-rate=96K \
  --turn-screen-off \
  --stay-awake \
  --keyboard=uhid \
  --mouse=uhid \
  --shortcut-mod=lalt \
  --window-title="Android [${DEVICE_IP}]"
```

### High-Fidelity Webcam Pipeline (`scrcpy-webcam.sh`)

```bash
#!/usr/bin/env bash
# Utilizes Android 12MP+ camera as a low-noise Linux/macOS V4L2 virtual camera
exec scrcpy \
  --video-source=camera \
  --camera-facing=back \
  --camera-size=1920x1080 \
  --camera-fps=60 \
  --no-audio \
  --v4l2-sink=/dev/video2 \
  --stay-awake
```

---

## 5. Diagnostic Matrix & Failure Modes

| Diagnostic Symptom | Root Cause | Definitive Engineering Remediation |
| :--- | :--- | :--- |
| **`device unauthorized`** | RSA key handshake pending user biometric/PIN verification. | Check device screen; toggle "Always allow from this computer" in USB Debugging modal. |
| **`MediaCodec video encoder error`** | SoC lacks hardware capability for requested resolution/codec. | Downgrade codec: `--video-codec=h264`, or constrain bounding box: `--max-size=1080`. |
| **Heavy lag / Packet loss over Wi-Fi** | High network jitter, 2.4GHz interference, or unconstrained bitrate. | Enforce 5GHz Wi-Fi band, clamp `--max-fps=60 -b 4M --max-size=1280`. |
| **Keystroke omission in non-English IME** | Default `sdk` input mode sends standard ASCII keycodes. | Enforce physical emulation: `--keyboard=uhid` or `--keyboard=aoa`. |
| **No audio output on stream** | Device runs `Android < 11` (AudioPlaybackCapture API unexposed). | Audio capture requires Android 11+; pass `--no-audio` on legacy operating systems. |
