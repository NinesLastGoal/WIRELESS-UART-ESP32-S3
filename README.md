Currently only the uart and wireless functions have been tested i havent finished the other features in their entirity and the logic analyzer is literally a placeholder lol. Uart works great though! I developed this to unfuck all those Chinese IOT devices so they can easily be reversed engineered. 





◉ ESP32-S3 UART Probe
A wireless UART pentesting tool built on the ESP32-S3-WROOM-1. Connect to target UART ports via WiFi and interact through a real-time web terminal with capture, logic analysis, and credential brute-forcing.

Chip	ESP32-S3 (QFN56) rev v0.2 — Dual-core 240 MHz
Flash	16 MB
PSRAM	8 MB Octal (capture buffer)
WiFi	2.4 GHz 802.11 b/g/n
USB Bridge	CH343 (programming & debug)
📌 Pin Wiring Guide
Connect these ESP32-S3 pins to your target device. All pins are 3.3 V logic — use a level shifter for 5 V targets.

Function	ESP32 GPIO	Board Pin	Connect To
UART TX	GPIO 17	Left side	Target RX
UART RX	GPIO 18	Left side	Target TX
Reset	GPIO 16	Left side	Target RST / EN
GND	GND	Both sides	Target GND
3.3V Out	3V3	Top-left	Target VCC (if needed)
⚠ TX/RX are crossed: Probe TX → Target RX, Probe RX ← Target TX. GND must always be shared.

🔍 Logic Analyzer Pins
Up to 4 channels. Any free GPIO can be used. Suggested GPIOs: 4, 5, 6, 7 (left side, near top). Resolution: 25 ns (40 MHz RMT sampling).

⛔ Restricted GPIOs (do NOT use): 26–34 (reserved for flash/PSRAM).

🚀 Quick Start
Wire GND, TX (GPIO 17), and RX (GPIO 18) to target.
Power on the probe. It creates WiFi AP: UART-PROBE (password: uartprobe).
Connect to the AP and open http://192.168.4.1 in your browser.
The Terminal tab shows live UART data via WebSocket.
Adjust baud rate in UART Config if needed, or use Auto-Detect Baud.
Use Reset Target to trigger boot output, then Boot Capture in Pentest to log it.
⚙ Feature Guide
Terminal	Real-time UART data over WebSocket. Type commands and send with Enter, CR, LF, or CRLF. Toggle hex view for binary protocols.
UART Config	Hot-swap baud rate, data bits, parity, stop bits, and mode (Active TX+RX or Sniff RX-only). Auto-Detect Baud measures signal edges to guess the correct rate.
Capture	7 MB ring buffer in PSRAM stores all UART traffic with timestamps. Download as binary for offline analysis.
Logic Analyzer	Capture digital signals on up to 4 GPIO channels using the RMT peripheral at 40 MHz. Useful for verifying wiring and protocol timing.
Pentest	Boot Capture: Reset + capture boot log. Shell Detection: Scan buffer for known prompts. Brute-Force: Try 2500+ default credentials against login prompts.
Files	Upload/download files to the on-board SPIFFS partition (12 MB). Store custom wordlists or captured data.
System	Live stats, WiFi STA config (connect to existing network for internet access while keeping AP), and reboot.
📡 API Endpoints
All endpoints return JSON. POST bodies are JSON. WebSocket at /ws for streaming.

Method	Path	Description
GET	/api/status	Device status (uptime, WiFi, UART, capture)
GET/POST	/api/uart/config	Get or set UART parameters
POST	/api/uart/send	Send data (text or hex)
POST	/api/uart/break	Send UART break signal
POST	/api/gpio/reset	Pulse target reset pin
POST	/api/gpio/set	Set any GPIO direction & level
POST	/api/autobaud/start	Start auto-baud scan
GET	/api/autobaud/result	Get scan results
GET	/api/capture/stats	Capture buffer statistics
GET	/api/capture/download	Download raw capture data
POST	/api/capture/clear	Clear capture buffer
POST	/api/logic/start	Start logic analyzer
POST	/api/logic/stop	Stop logic analyzer
POST	/api/pentest/boot-capture	Reset + capture boot output
POST	/api/pentest/brute	Start credential brute-force
GET	/api/pentest/brute/status	Brute-force progress
GET	/api/files/list	List SPIFFS files
POST	/api/files/upload?name=X	Upload file
GET	/api/files/download?name=X	Download file
DELETE	/api/files/delete?name=X	Delete file
POST	/api/wifi/config	Set STA credentials
GET	/api/wifi/status	WiFi status
POST	/api/system/reboot	Reboot device
WS	/ws	WebSocket (binary frames = UART RX)
