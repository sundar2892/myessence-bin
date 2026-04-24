# MYESSENCE ESP32 — Testing guide (step by step)

How to verify **Wi‑Fi provisioning**, **web setup**, **toy I2S audio**, **gateway WebSocket**, and **buttons** using this firmware.

---

## 1. Prerequisites

1. Firmware and LittleFS flashed (**`upload`** + **`uploadfs`**) or a full merged image flashed once. See [Firmware-setup.md](Firmware-setup.md).
2. Hardware wired per [Hardware-setup.md](Hardware-setup.md).
3. Serial monitor at **921600** baud to read logs (`pio device monitor -b 921600`).

---

## 2. First boot — no saved Wi‑Fi (captive portal / AP mode)

1. If **no SSID** is stored in NVS, the device starts **SoftAP**:
   - SSID (example from firmware): **`BUDDYBEAR MYESSENCE`**
   - AP IP: **`192.168.4.1`**
2. On your phone or PC, **join that Wi‑Fi network**.
3. Open a browser:

   **`http://192.168.4.1/`**

   You should see **`configuration.html`** (MYESSENCE setup wizard).

---

## 3. Web setup wizard (recommended order)

The page uses **three steps**:

### Step 1 — WiFi & WebSocket

1. Enter **home Wi‑Fi SSID** and **password** (the network the toy will join after reboot).
2. Enter **WebSocket URL** (must start with `ws://` or `wss://` — same validation as firmware).
3. Tap **Next: test toy audio**.

### Step 2 — Toy speaker & microphone (ESP32 I2S)

These calls hit the device over HTTP (not your laptop mic):

1. **Speaker:** **Play chime on toy** → `POST /hardware-audio/speaker` → listen at the **MAX98357** speaker. Then **I heard it**.
2. **Microphone:** **Start mic level** → page polls `GET /hardware-audio/mic` → speak near **INMP441**; the bar should move. Then **Stop — mic OK**.
3. Optional: **Skip toy audio** if I2S is disabled in build or you are debugging Wi‑Fi only.
4. Tap **Next: connect**.

### Step 3 — Save & finish

1. Review the short text; tap **Connect** to **`POST /connect`** (saves NVS: SSID, password, socket URL).
2. On success, tap **Go to home page** → opens **`./myessence-home.html`** from LittleFS.
3. The device **restarts** a few seconds after save (firmware behavior). After reboot, connect your phone/PC to **your home Wi‑Fi** (not the AP) to reach the toy’s new LAN IP if you open services by IP.

---

## 4. After reboot — STA mode (home Wi‑Fi)

1. Watch **serial** for a line like: Wi‑Fi **connected** and **IP=…**.
2. Gateway WebSocket:
   - On success you should see activity such as **websocket opened, waiting for hello_ack** and later **`hello_ack`** handling (RGB may show “ready” patterns depending on firmware).
   - **`CONNECT FAILED state=-2`** means the TCP/WebSocket **`connect()`** failed (wrong URL, DNS, firewall, TLS, or server down). Fix the **saved socket URL** (re-enter AP mode by clearing Wi‑Fi / erasing flash, or change NVS via your process).

---

## 5. Quick API checks (optional)

While connected to the **setup AP** (`192.168.4.1`):

| Check | Method | Expected |
|--------|--------|----------|
| Speaker test | `POST http://192.168.4.1/hardware-audio/speaker` | HTTP 200, JSON `{"ok":true}` |
| Mic level | `GET http://192.168.4.1/hardware-audio/mic` | HTTP 200, JSON `{"ok":true,"rms":…}` |

Use **curl**, **Postman**, or the setup page buttons.

---

## 6. Physical interaction tests

| Test | Action | Expected (when gateway is connected and firmware logic allows) |
|--------|--------|-------------------------------------------------------------------|
| **Wake** | Hold **GPIO12** to GND **~5 s** | Wake phrase sent (see serial / backend) |
| **Hold-to-record** | Hold **GPIO14** to GND **≥ ~300 ms** | Mic streaming to gateway over WebSocket (RGB / logs) |

Short taps on **GPIO14** should **not** start streaming (hold debounce).

---

## 7. RGB LED (rough meaning)

LED patterns depend on `rgb_led` code; in general:

- **Connecting Wi‑Fi** / **connecting gateway** / **link lost** use different colors or animations.
- Use serial tags like **`WIFI`**, **`SOCKET`**, **`RGB`** to correlate behavior.

---

## 8. Common issues

| Symptom | What to check |
|---------|----------------|
| **Cannot open `192.168.4.1`** | Join **`BUDDYBEAR MYESSENCE`** AP; try `http://` not `https://` for captive portal. |
| **No config page** | Reflash **LittleFS** (`uploadfs`); confirm `data/configuration.html` exists. |
| **Speaker/mic test HTTP 503** | Build may have **`MYESSENCE_I2S_SPEAKER=0`**; use **Skip toy audio** or enable I2S in build. |
| **`CONNECT FAILED state=-2`** | Socket URL, TLS (`wss://`), network egress, server WebSocket path. |
| **No audio on speaker** | Wiring for BCLK/WS/DOUT; try `MYESSENCE_I2S_SPEAKER_RIGHT_CHANNEL` (see Hardware doc). |

---

## 9. Related docs

- **Pins and wiring:** [Hardware-setup.md](Hardware-setup.md)  
- **Build, flash, merged BIN:** [Firmware-setup.md](Firmware-setup.md)
