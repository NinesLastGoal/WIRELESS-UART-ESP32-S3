# ◉ ESP32-S3 UART Probe

Built to tackle the endless parade of locked-down IoT devices flooding the market. This one runs on an ESP32-S3-WROOM-1 and lets you tap into UART ports wirelessly—full terminal access, signal capture, and some useful reverse engineering tools bundled in.

**Status:** UART and WiFi are solid. Brute-force and boot capture work. Logic analyzer is still under construction—don't rely on it yet.

## Hardware Specs

| Component | Details |
|-----------|---------|
| **Chip** | ESP32-S3 (QFN56) rev v0.2 — Dual-core @ 240 MHz |
| **Flash** | 16 MB |
| **PSRAM** | 8 MB Octal (used for capture ring buffer) |
| **WiFi** | 2.4 GHz 802.11 b/g/n |
| **USB Bridge** | CH343 (programming & debug) |

## 📌 Wiring

Connect to your target's UART. Everything runs on 3.3V logic—use a level shifter if the target is 5V.

| Function | ESP32 GPIO | Board Pin | → Target |
|----------|-----------|-----------|----------|
| TX | GPIO 17 | Left side | RX |
| RX | GPIO 18 | Left side | TX |
| RST | GPIO 16 | Left side | RST / EN |
| GND | GND | Both sides | GND |
| 3.3V Out | 3V3 | Top-left | VCC (optional) |

**Remember:** TX and RX are crossed. GND must be shared or nothing works.

## 🔍 Logic Analyzer (WIP)

Up to 4 channels on any free GPIO. Suggested: 4, 5, 6, 7 (left side, near top). Resolution is 25 ns (40 MHz RMT sampling).

⛔ **Don't use GPIOs 26–34** — they're tied to flash and PSRAM.

## 🚀 Quick Start

1. Wire up GND, TX (GPIO 17), and RX (GPIO 18) to your target
2. Power on the probe—it'll broadcast a WiFi AP named `UART-PROBE` (password: `uartprobe`)
3. Connect to it, open http://192.168.4.1
4. Terminal tab shows live UART data. Adjust baud rate or use Auto-Detect
5. For boot logs, hit "Reset Target" then "Boot Capture" in the Pentest tab

## ⚙️ Features

**Terminal** — Real-time UART console over WebSocket. Send text, hex, or raw bytes. Supports CR, LF, CRLF.

**UART Config** — Swap baud rates on the fly, tweak data bits, parity, stop bits. Auto-Detect tries to figure out the speed by measuring signal edges.

**Capture** — 7 MB ring buffer in PSRAM grabs everything with timestamps. Download as binary for offline analysis.

**Logic Analyzer** — Digital signal capture on 4 GPIO channels @ 40 MHz. Still iterating on this one.

**Pentest** — Boot capture (reset + log), shell detection (hunts for common prompts), and 2500+ default credential brute-force.

**Files** — Upload/download custom wordlists or captured data to the on-board SPIFFS (12 MB).

**System** — Status, WiFi config (can stay in AP mode and bridge to another network), reboot.

## 📡 API Endpoints

All responses are JSON. POST bodies are JSON. WebSocket streaming at `/ws`.

| Method | Path | What it does |
|--------|------|--------------|
| GET | `/api/status` | Device uptime, WiFi, UART, capture stats |
| GET/POST | `/api/uart/config` | Get or set UART parameters |
| POST | `/api/uart/send` | Send text or hex data |
| POST | `/api/uart/break` | Send UART break signal |
| POST | `/api/gpio/reset` | Pulse the reset pin |
| POST | `/api/gpio/set` | Set GPIO direction & level |
| POST | `/api/autobaud/start` | Start baud rate detection |
| GET | `/api/autobaud/result` | Get detection results |
| GET | `/api/capture/stats` | Buffer usage & metadata |
| GET | `/api/capture/download` | Download raw capture |
| POST | `/api/capture/clear` | Wipe the buffer |
| POST | `/api/logic/start` | Start logic analyzer |
| POST | `/api/logic/stop` | Stop logic analyzer |
| POST | `/api/pentest/boot-capture` | Reset device + capture boot |
| POST | `/api/pentest/brute` | Start credential brute-force |
| GET | `/api/pentest/brute/status` | Check progress |
| GET | `/api/files/list` | List SPIFFS files |
| POST | `/api/files/upload?name=X` | Upload file |
| GET | `/api/files/download?name=X` | Download file |
| DELETE | `/api/files/delete?name=X` | Delete file |
| POST | `/api/wifi/config` | Set WiFi STA credentials |
| GET | `/api/wifi/status` | WiFi status |
| POST | `/api/system/reboot` | Reboot device |
| WS | `/ws` | WebSocket for UART RX (binary frames) |