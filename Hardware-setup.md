# MYESSENCE ESP32 — Hardware setup (step by step)

This guide matches the pin definitions in `include/config.h` and the **ESP32-S3-DevKitC-1** (8 MB flash, no PSRAM) target in `platformio.ini`.

---

## 1. What you need

| Item | Role |
|------|------|
| **ESP32-S3-DevKitC-1** (or compatible S3 board, **8 MB** flash) | Main MCU |
| **INMP441** (or compatible I2S mic) | Microphone → ESP32 |
| **MAX98357A** (or compatible I2S amp + speaker) | Speaker ← ESP32 |
| **NeoPixel / WS2812** on GPIO **48** (often already on DevKitC-1) | Status LED |
| USB cable (data-capable) | Power + serial + flash |
| Jumper wires, solid grounds, stable 3.3 V | Reliable audio |

---

## 2. Power and ground

1. Connect **ESP32 3.3 V** and **GND** to each breakout that needs logic power (mic board, amp if required by your module).
2. Use a **common ground** between ESP32, INMP441, and MAX98357.
3. Prefer **short wires** for I2S (especially BCLK / WS / data).

---

## 3. I2S speaker (MAX98357A)

Wire the speaker module to the **same I2S bus** pins used for TX in firmware:

| Signal | ESP32 GPIO | Notes |
|--------|------------|--------|
| **BCLK** | **5** | Bit clock |
| **LRCK / WS** | **4** | Word select |
| **DIN** (data **into** amp) | **6** | I2S data out from ESP32 |

**Also:**

- Follow your MAX98357 breakout for **VIN**, **GND**, and **speaker outputs** to the speaker.
- **Gain / SD** pins are often fixed on the breakout; they are not assigned in `config.h`.

If audio is **very quiet, noisy, or only one channel wrong**, you can try changing **`MYESSENCE_I2S_SPEAKER_RIGHT_CHANNEL`** to `1` in `platformio.ini` build flags (see comments in `config.h`).

---

## 4. I2S microphone (INMP441)

| Signal | ESP32 GPIO | Notes |
|--------|------------|--------|
| **SD / DOUT** (data **from** mic) | **7** | Must match `kI2sMicDinPin` |
| **SCK** (BCLK) | **5** | Same BCLK as speaker (duplex I2S) |
| **WS** | **4** | Same WS as speaker |
| **VDD / GND** | 3.3 V / GND | Per INMP441 module |

The firmware uses **duplex I2S** (TX + RX on one port). **Do not** share GPIOs with conflicting functions.

---

## 5. Buttons (digital inputs, pull-ups)

| Function | GPIO | Behavior |
|----------|------|----------|
| **Wake / general button** | **12** | Hold **GPIO12 to GND** for **5 s** (`kWakeHoldMs`) to send wake phrase |
| **Hold-to-record** | **14** | Hold **GPIO14 to GND** **≥ 300 ms** before streaming starts (short taps ignored) |

**Important:** **GPIO14** must **not** be the same pin as **I2S BCLK (5), WS (4), DOUT (6), or mic DIN (7)**. If you move I2S, update `config.h` and keep the record pin free.

Wiring: one side of the button to the GPIO, other side to **GND** (internal pull-up is used).

---

## 6. Onboard RGB (WS2812)

- Default: **GPIO 48**, **1** LED, brightness cap in `config.h` (`kRgbBrightness`).
- If your board revision uses a different pin for the onboard LED, change **`kRgbPin`** in `config.h`.

---

## 7. USB serial

- `platformio.ini` sets **921600** baud for the monitor.
- `ARDUINO_USB_CDC_ON_BOOT` is enabled: on many S3 boards, **Serial** is the USB CDC port.

---

## 8. After wiring — quick checklist

1. **No shorts** between 3.3 V and GND.
2. **BCLK / WS** shared correctly between mic and amp.
3. **DOUT (speaker)** on **6**, **DIN (mic)** on **7**.
4. **GPIO12 / GPIO14** buttons only to GND when pressed.
5. Flash size **8 MB** if you use the provided merged image (`myessence-fullflash-8mb.bin`).

---

## 9. Related docs

- **Build, flash, merged BIN:** [Firmware-setup.md](Firmware-setup.md)  
- **Provisioning, web UI, serial checks:** [Testing-guide.md](Testing-guide.md)
