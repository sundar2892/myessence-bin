# MYESSENCE ESP32 — Firmware setup (step by step)

Target: **PlatformIO**, board **`esp32-s3-devkitc-1`**, environment **`esp32s3`**, **LittleFS** for files in `data/`.

**Two ways to put firmware on the board**

| Method | When to use |
|--------|-------------|
| **PlatformIO** `upload` + `uploadfs` | Day-to-day development over USB (sections 3–4). |
| **Espressif Flash Download Tool** | Factory-style flashing, merged **one-file** image, or when you prefer the GUI (section 8). |

You still use PlatformIO on the PC to **build** the `.bin` files; the Flash Download Tool only **writes** them to flash.

---

## 1. Install software

1. Install **Visual Studio Code**.
2. Install the **PlatformIO** extension.
3. Open the folder: **`my-essence-ai-brain/esp32`** (the folder that contains `platformio.ini`).

---

## 2. First-time build

1. Open a **PlatformIO terminal** (or any terminal) in the **`esp32`** directory.
2. Run:

   ```bash
   pio run -e esp32s3
   ```

3. Wait until you see **SUCCESS**. This downloads libraries and compiles the firmware.

---

## 3. Flash firmware (USB upload)

1. Connect the ESP32-S3 with a **data** USB cable.
2. If upload fails, enter **download mode**: hold **BOOT**, tap **RESET**, release **BOOT**.
3. In the project folder:

   ```bash
   pio run -e esp32s3 -t upload
   ```

4. When finished, reset the board.

---

## 4. Flash LittleFS (web pages in `data/`)

The config UI (`configuration.html`, CSS, images) lives in **`data/`** and must be uploaded separately:

```bash
pio run -e esp32s3 -t uploadfs
```

Do this whenever you change files under **`data/`**.

---

## 5. Serial monitor

Baud rate is **921600**:

```bash
pio device monitor -b 921600
```

Or:

```bash
pio run -e esp32s3 -t monitor
```

---

## 6. Clean build (PC only — does not erase the chip)

```bash
pio run -e esp32s3 -t clean
pio run -e esp32s3
```

---

## 7. Erase entire flash (factory wipe on the chip)

Removes firmware, filesystem, and saved Wi‑Fi/socket settings:

```bash
pio run -e esp32s3 -t erase
```
//////////////////////////







Then flash again with **`upload`** and **`uploadfs`** (or use your merged BIN; see below).

---

## 8. **Espressif Flash Download Tool** (GUI) — 8 MB merged image

Use this when you flash with **Espressif’s Flash Download Tools** (Windows GUI), not `pio upload`.

Download the tool from Espressif: **[Other tools → Flash Download Tools](https://www.espressif.com/en/support/download/other-tools)** (pick the package that includes **Flash Download Tool** for your OS; Windows is most common).

### 8.1 Build the merged `.bin` on your PC

The GUI needs **one file** at **`0x0`**: bootloader + partition table + `boot_app0` + firmware + LittleFS merged in order. This repo ships a script that does that after a normal PlatformIO build.

1. Install **VS Code + PlatformIO** and open the **`esp32`** folder (section 1).
2. Open **PowerShell**, go to the firmware directory:

   ```powershell
   cd path\to\my-essence-ai-brain\esp32
   ```

3. Run:

   ```powershell
   powershell -ExecutionPolicy Bypass -File .\scripts\merge_fullflash_8mb.ps1
   ```

   This runs `pio run`, builds the LittleFS image, then **`esptool merge-bin`** into a single file.

4. Output file (copy this path into the Flash Download Tool):

   **`esp32\.pio\build\esp32s3\myessence-fullflash-8mb.bin`**

   (Path is under your project; `.pio` is not in Git.)

Run the script again after you change application code **or** anything under **`data/`** (web UI), so the merged image stays current.

### 8.2 Open the Flash Download Tool and select the chip

1. Launch **flash_download_tool_3.x.x.exe** (exact name varies by package).
2. When asked for chip/work mode, choose **ESP32-S3** and the mode that matches **UART** flashing over USB (often **develop** / factory — use the option that lists **COM** ports for download).
3. In the main window, set **SPI SPEED** to **80 MHz**, **SPI MODE** to **DIO**, **FLASH SIZE** to **8 MByte** (matches **ESP32-S3-DevKitC-1 N8** and this project’s merge script).

### 8.3 Add the merged file (single row)

1. In the file list, enable **one** row (checkbox on).
2. **Address:** `0x0`
3. **Path:** browse to **`myessence-fullflash-8mb.bin`** (the file built in step 8.1).

Do **not** add separate `bootloader.bin`, `partitions.bin`, etc., when using this merged image — everything is already inside the one file at `0x0`.

### 8.4 COM port and download mode

1. Plug the board in with a **data** USB cable.
2. Choose the correct **COM** port (Device Manager → “USB Serial” / “COMx” for your board).
3. If the tool does not connect, put the chip in **download / UART boot** mode: hold **BOOT**, tap **RESET**, release **BOOT**, then click **START** in the tool.

### 8.5 Flash and reboot

1. Click **START** and wait until the log shows success.
2. Press **RESET** on the board (or power-cycle) so it runs from flash, not the bootloader.

### 8.6 Notes and limits

- Valid only for **8 MB** flash and the **default `default_8MB`** partition layout used by **`esp32-s3-devkitc-1`** in this `platformio.ini`. If you change the board or partition table, you must change **`scripts\merge_fullflash_8mb.ps1`** offsets and rebuild the merge.
- If you prefer **not** using a merged file, you can flash five binaries at five addresses instead; the merge script documents the same offsets (`0x0`, `0x8000`, `0xe000`, `0x10000`, `0x670000`) — but one file at `0x0` is simpler and less error-prone.

---

## 9. If `merge-bin` fails (esptool version)

Some installs use the older command name. In `scripts\merge_fullflash_8mb.ps1`, replace **`merge-bin`** with **`merge_bin`** if your `python -m esptool` reports an unknown command.

---

## 10. Default WebSocket URL (source code)

The default gateway URL is set in **`include/config.h`** (`kDefaultSocketUrl`). Saved URL is overridden by **NVS** after you use the captive portal **Connect** flow. See [Testing-guide.md](Testing-guide.md).

---

## 11. Related docs

- **Wiring:** [Hardware-setup.md](Hardware-setup.md)  
- **Provisioning and tests:** [Testing-guide.md](Testing-guide.md)
