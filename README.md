# MYESSENCE ESP32 — documentation

This folder is the **setup and testing handbook** for the MYESSENCE firmware that runs on **ESP32-S3-DevKitC-1** (8 MB flash, PlatformIO environment `esp32s3`, LittleFS for the web UI in `data/`). Application source code lives in the firmware repo (for example **`my-essence-ai-brain/esp32`**, where `platformio.ini` lives); these files describe how to wire the board, build and flash, and verify behavior end to end.

---

## Guides

| Document | What it covers |
|----------|----------------|
| [Firmware-setup.md](Firmware-setup.md) | VS Code + PlatformIO, first build, USB `upload` / `uploadfs`, serial monitor (921600), clean and full erase, **Espressif Flash Download Tool** with merged 8 MB image (`merge_fullflash_8mb.ps1`), `esptool merge-bin` notes, default WebSocket URL in `include/config.h`. |
| [Hardware-setup.md](Hardware-setup.md) | Bill of materials (S3, INMP441, MAX98357A, WS2812 on GPIO 48), power and ground, **duplex I2S** pin table (BCLK 5, WS 4, speaker DIN 6, mic DOUT 7), wake GPIO12 and record GPIO14, USB serial, post-wiring checklist. |
| [Testing-guide.md](Testing-guide.md) | First boot captive portal (**`BUDDYBEAR MYESSENCE`** / `192.168.4.1`), three-step web wizard (Wi‑Fi + WebSocket, toy I2S tests, save via `/connect`), STA mode and gateway WebSocket logs, optional HTTP checks, button and RGB behavior, common issues. |

Read **Hardware** before soldering, **Firmware** before flashing, **Testing** after the device is on the network you care about.

---

## At a glance

- **Board:** ESP32-S3-DevKitC-1, **8 MB** flash (matches merged full-flash image and partition layout).
- **Flash firmware + filesystem:** `pio run -e esp32s3 -t upload` then `pio run -e esp32s3 -t uploadfs` whenever `data/` changes.
- **Serial:** `pio device monitor -b 921600` (or PlatformIO monitor target).
- **Factory wipe chip:** `pio run -e esp32s3 -t erase`, then upload again.
- **Config UI:** served from LittleFS; default socket URL is in `include/config.h` until overridden by NVS after the portal **Connect** flow.

For step-by-step commands, addresses, and troubleshooting tables, use the linked guides above.
