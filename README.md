# u-connectMatter

**A binary-distribution, ready-to-run Matter end-node reference from u-blox.**

[![License](https://img.shields.io/badge/license-Apache--2.0-blue)](https://www.apache.org/licenses/LICENSE-2.0)
[![Matter SDK](https://img.shields.io/badge/Matter%20SDK-1.5.1.0-brightgreen)](https://github.com/project-chip/connectedhomeip)
[![Platforms](https://img.shields.io/badge/platforms-Windows%20%7C%20Linux%20%7C%20STM32%20%7C%20Pico-lightgrey)](https://github.com/u-blox/u-connectMatter/releases)
[![Latest release](https://img.shields.io/github/v/release/u-blox/u-connectMatter?display_name=tag&sort=semver&label=download)](https://github.com/u-blox/u-connectMatter/releases/latest)

> **Status:** This repository hosts **prebuilt, signed release binaries** of the
> u-connect&nbsp;Matter reference application. The source tree is not (yet)
> mirrored here — see [Releases](https://github.com/u-blox/u-connectMatter/releases)
> for downloads or contact u-blox for source access.

> **NORA-W36 firmware requirement:** Matter requires **u-connect&nbsp;Express
> firmware 3.4.0 or newer** on the NORA-W36 module (earlier firmware lacks the
> platform mDNS + IPv6 multicast features Matter depends on). Check your
> module's version with `AT+GMR` and update via the
> [u-connectXpress NORA-W36 release notes](https://github.com/u-blox/u-connectXpress/tree/main/NORA-W36)
> if needed. The app checks the firmware version at startup and can update the
> module itself — see [Updating NORA-W36 firmware](#updating-nora-w36-firmware).

---

## What is u-connectMatter?

A Matter 1.5.1 end-node reference application that runs on a wide range of host
platforms, using the **u-blox NORA-W36** module for dual-band Wi-Fi 4
(2.4 + 5&nbsp;GHz) and Bluetooth&nbsp;LE 5.3 connectivity. It demonstrates how to ship a CSA-certifiable Matter product on
top of u-connect&nbsp;Express firmware with minimal host-side code.

The release binaries run the **multi-endpoint combo demo** — one Matter node
exposing a full house of virtual devices you can drive from the keyboard:

- **Color Lights** ×3 + **Dimmable Light** (On/Off + Level Control)
- **Door Lock**
- **Light Switch** (action button)
- **Temperature / Humidity / Occupancy / Contact / Soil-moisture sensors**
- **Thermostat** (interactive)
- **Water Leak Detector**, **Smoke/CO Alarm**
- **Doorbell Chime**, **Window Covering**, **Fan**

Commissioning is the standard Matter flow: scan the QR code from the device's
serial console (or web dashboard) with the Apple Home / Google Home / Amazon
Alexa / SmartThings app.

---

## Supported targets

All host platforms get the same wireless capability from the NORA-W36 module:
dual-band Wi-Fi 4 (2.4 + 5&nbsp;GHz, 802.11 a/b/g/n) and Bluetooth&nbsp;LE 5.3.

| Host platform | Hardware | Artifact |
|---|---|---|
| **Windows 10/11 x64** | PC + NORA-W36 EVK (USB-UART) | signed `.exe` |
| **Linux x86_64** | PC / SBC + NORA-W36 EVK | `.tar.gz` |
| **STM32 H7** (H753, H743, H735, H723, H750) | NUCLEO / Discovery + NORA-W36 | `.bin` / `.hex` / `.elf` |
| **STM32 F4** (F407, F429, F439) | NUCLEO + NORA-W36 | `.bin` / `.hex` / `.elf` |
| **Raspberry Pi Pico** (RP2040) | Pico + NORA-W36 | `.uf2` |
| **Raspberry Pi Pico 2** (RP2350) | Pico 2 + NORA-W36 | `.uf2` |

Every release artifact ships with a SHA-256 checksum sidecar
(`<file>.sha256`). The Windows `.exe` is Authenticode-signed.

---

## Wiring &mdash; how to connect NORA-W36 to each host

For **Windows / Linux PCs** there is no wiring: the **EVK-NORA-W36** evaluation
kit exposes the module's UART through its on-board USB-UART bridge. Plug in the
USB cable, the host enumerates a virtual COM port (`COMxx` on Windows,
`/dev/ttyUSB0` on Linux), and the app auto-detects it.

For the **STM32** and **Pico** targets the host's UART is wired directly to
NORA-W36's UART. Cross TX&harr;RX, share GND, supply 3.3&nbsp;V. Hardware flow
control (CTS/RTS) is **not** required &mdash; the firmware runs reliably without
it on all supported boards.

| Host board | Host UART | Host TX&nbsp;&rarr;&nbsp;NORA&nbsp;RX | Host RX&nbsp;&larr;&nbsp;NORA&nbsp;TX | Baud | Notes |
|---|---|---|---|---|---|
| **NUCLEO-H753ZI** | USART1 | **PB6** &mdash; Morpho CN7 pin&nbsp;1 | **PB7** &mdash; Morpho CN7 pin&nbsp;21 | 1&nbsp;Mbit/s | High-speed; CTS/RTS available on PA11/PA12 but unused |
| **NUCLEO-H743ZI** | UART4 | **PA0** &mdash; CN11 pin&nbsp;28 | **PA1** &mdash; CN11 pin&nbsp;30 | 1&nbsp;Mbit/s | No HW flow control |
| **NUCLEO-H723ZG** | UART4 | **PA0** | **PA1** | 1&nbsp;Mbit/s | No HW flow control |
| **NUCLEO-H735IG** / Disco | USART1 | **PA9** &mdash; Arduino D8 (CN12) | **PA10** &mdash; Arduino D7 (CN12) | 1&nbsp;Mbit/s | |
| **NUCLEO-H750ZB** | UART4 | **PA0** | **PA1** | 1&nbsp;Mbit/s | No HW flow control |
| **NUCLEO-F407** (Disco / dev board) | USART1 | **PA9** | **PA10** | 1&nbsp;Mbit/s | |
| **NUCLEO-F429ZI** | USART1 | **PA9** | **PA10** | 1&nbsp;Mbit/s | |
| **NUCLEO-F439ZI** | USART1 | **PA9** | **PA10** | 1&nbsp;Mbit/s | |
| **Raspberry Pi Pico** (RP2040) | UART0 | **GP0** &mdash; pin&nbsp;1 | **GP1** &mdash; pin&nbsp;2 | 115&nbsp;200 | |
| **Raspberry Pi Pico 2** (RP2350) | UART0 | **GP0** &mdash; pin&nbsp;1 | **GP1** &mdash; pin&nbsp;2 | 115&nbsp;200 | |

**Always required, every wired target:**
- **GND** &harr; NORA-W36 **GND**
- **3.3&nbsp;V** &rarr; NORA-W36 **VCC** (do **not** use 5&nbsp;V; NORA-W36 is 3V3-only)
- (recommended) NORA-W36 **RESET_N** to a host GPIO so the launcher can hard-reset the module

Debug output (Matter logs, QR code) is printed on a **second UART** on the host:
ST-Link Virtual COM Port on all NUCLEO boards (USART3 PD8/PD9 on H7/F439, USART2
PA2/PA3 on F407/F429) and the Pico's USB serial. No extra wiring needed for the
debug console.

NORA-W36 module pinout, electrical characteristics and EVK schematics are in the
[NORA-W36 data sheet](https://www.u-blox.com/en/product/nora-w36-series) on the
u-blox product page.

---

## Quick start (Windows)

1. Download the latest **`u-connect-matter-windows-x64-<version>.exe`** from
   [Releases](https://github.com/u-blox/u-connectMatter/releases/latest).
2. Plug in your **EVK-NORA-W36** evaluation kit via USB.
3. Run the `.exe`. The app auto-detects the EVK's COM port, opens the serial
   link to the module, starts the Matter stack, and prints the commissioning
   QR code on the console. On first run it also writes a starter
   `matter.config` next to the `.exe`.
4. Scan the QR code from your phone's Matter-compatible smart-home app.

Press `h` (or `?`) at any time for the interactive keyboard command list, and
`w` to start the built-in **web dashboard** — served by the NORA-W36 module at
`http://<device-ip>/` (port 80) once Wi-Fi is connected — showing live link
state, fabric/subscription health, and current cluster values.

## Quick start (STM32 NUCLEO + NORA-W36)

1. Wire NORA-W36 EVK UART to the NUCLEO header (RX/TX + 3V3 + GND, see table above).
2. Drag-and-drop the matching `.bin`/`.hex` for your board (e.g.
   `u-connect-matter-stm32-h753-<version>.bin`) onto the NUCLEO's mass-storage
   programmer, or flash with STM32CubeProgrammer / `st-flash`.
3. Open the NUCLEO virtual COM port at 115200 8N1 to see the QR code.

## Quick start (Raspberry Pi Pico)

1. Hold BOOTSEL while plugging in the Pico/Pico 2.
2. Drag `u-connect-matter-pico-<version>.uf2` (or `pico2`) onto the
   `RPI-RP2` / `RP2350` mass-storage drive.
3. Open the Pico's USB serial port to see the QR code.

---

## Command-line arguments (Windows / Linux)

```
u-connect-matter [options] [COMxx | /dev/ttyXXX] [device-type]
```

All arguments are optional — with no arguments the app auto-detects the serial
port, loads `matter.config`, and starts the Matter stack. Command-line
arguments override the corresponding `matter.config` settings.

| Argument | Description |
|---|---|
| `--port <port>` | Serial port of the NORA-W36 module (e.g. `--port COM7`, `--port /dev/ttyUSB0`). A bare `COMxx` or `/dev/...` positional argument works too. Default: auto-detect. |
| `--dev` | Developer mode — auto-connects Wi-Fi using the `WIFI_PROFILE_*` credentials in `matter.config` (skips BLE commissioning of the network). |
| `--verbose` | Full Matter SDK log stream on the console. By default the console is in *simple* mode: banners + a compact 12-step commissioning card only. The file log records everything either way. |
| `--simple` | Simple console mode (explicit; already the default). |
| `--no-matter` | Skip the Matter server — Wi-Fi + web dashboard only (diagnostics). |
| `--test` | Test mode — BLE + UDP echo responder for automated test scripts. |
| `--ftxui` | Experimental FTXUI live text-UI dashboard. |
| `<device-type>` | Device personality for single-device builds: `light` (default), `lock`, `switch`, `temperature`, `waterleak`, `contact`, `chime`, `soil`, `window`, `fan`, `smoke`, `dimmable`. The standard release binary is the multi-endpoint **combo** build, where all device types are always present. |

Examples:

```powershell
# Auto-detect everything (typical)
.\u-connect-matter-windows-x64-3.4.7.exe

# Explicit COM port + auto Wi-Fi connect from matter.config
.\u-connect-matter-windows-x64-3.4.7.exe --port COM31 --dev

# Full SDK logging for debugging
.\u-connect-matter-windows-x64-3.4.7.exe --verbose
```

## Configuration — `matter.config`

All user-editable settings live in a plain-text `matter.config` file next to
the executable (`KEY=VALUE`, `#` comments). A fully-commented template is
included in this repository: [matter.config](matter.config). On first run the
app writes a starter file automatically. Highlights:

| Key | Purpose | Default |
|---|---|---|
| `UCM_COM_PORT` | Serial port, or `AUTO` for auto-detection | `AUTO` |
| `UCM_BAUD_RATE` | UART baud after handshake (115200 / 921600 / 1000000 / 3000000) | `1000000` |
| `UCM_FLOW_CONTROL` | CTS/RTS hardware flow control | `false` |
| `UCM_REG_DOMAIN` | Wi-Fi regulatory domain (0=World, 1=ETSI, 2=FCC, …) | `0` |
| `UCM_CHANNEL_LIST` | Restrict Wi-Fi channels (`5g`, `2.4g`, `1,6,11`, …) | full regdomain |
| `WIFI_PROFILE_0_SSID` / `_PASSWORD` | Wi-Fi credentials (profiles 0–2) used by `--dev` mode | — |
| `STATIC_IP` / `STATIC_MASK` / `STATIC_GATEWAY` / `STATIC_DNS` | Static IP (DHCP if unset) | DHCP |
| `MATTER_VENDOR_ID` / `MATTER_PRODUCT_ID` | Matter identity (replace test IDs for certification) | `0xFFF1` / `0x8000` |
| `MATTER_DISCRIMINATOR` / `MATTER_PASSCODE` | Commissioning discriminator & passcode | `2736` / `36363636` |
| `MATTER_DEVICE_NAME` | Name shown in controller apps | `ucx-matter-app` |
| `DEV_MODE` / `TEST_MODE` / `WEB_SERVER` / `AT_LOG` / `VERBOSE_LOGGING` | Persistent equivalents of the CLI flags / runtime toggles | `false`/`false`/`false`/`true`/`false` |

Other files created in the working directory: `matter_kvs.bin` (Matter fabrics,
ACLs, sessions — wiped by factory reset), `matter_settings.bin` (app
preferences) and `ucx-matter-app.log` (full application log, overwritten each
start).

## Keyboard commands

Press `h` or `?` in the console for the complete, build-specific list. The most
useful ones:

| Key | Action |
|---|---|
| `1`–`3` | Toggle Color Lights (endpoints 1–3) |
| `4` / `5` | Toggle Dimmable / Simple Light |
| `6` `7` `8` `9` `0` `-` `=` `+` | Doorbell, soil sensor, occupancy, window covering, fan, smoke/CO alarm, brightness, water leak |
| `c` | Open/close the commissioning window |
| `d` | Dashboard — Wi-Fi, Matter fabric and device status |
| `w` | Start/stop the web dashboard (`http://<device-ip>/`) |
| `a` | Toggle AP provisioning (captive portal for Wi-Fi setup) |
| `l` / `u` / `s` | Toggle verbose Matter log / AT command log / simple console |
| `n` | NTP time sync |
| `x` | XMODEM firmware update menu (NORA-W36) |
| `!` | **Factory reset** — clears all fabrics, ACLs and the module's Wi-Fi credentials, then exits |
| `q` | Quit |

## Updating NORA-W36 firmware

The app requires u-connect&nbsp;Express **3.4.0+** and checks at startup. To
update, drop the firmware file (e.g. `NORA-W36X-SW-3.4.0-056.bin` or the
release `.zip`) next to the executable (or in a `firmware/` subfolder) and
start the app — it detects the newer firmware, asks `[y/N]`, and flashes the
module over XMODEM. After a successful update the file is moved to
`firmware/archive/`. You can also trigger the menu manually with the `x` key.

---

## Hardware: NORA-W36

[NORA-W36](https://www.u-blox.com/en/product/nora-w36-series) is a u-blox
short-range module providing **dual-band Wi-Fi 4** (IEEE&nbsp;802.11&nbsp;a/b/g/n,
2.4 + 5&nbsp;GHz) and **Bluetooth&nbsp;LE 5.3** in a single
14.3 &times; 10.4 &times; 1.9&nbsp;mm SMD package. In this application NORA-W36
runs u-connect&nbsp;Express firmware (`AT`-controlled offload engine) so any
host MCU &mdash; from an STM32 F407 to a desktop PC &mdash; can run a full
Matter end-node by just driving a UART.

The host-side glue is provided by
[u-connectClient](https://github.com/u-blox/u-connectClient), the open-source
u-connect&nbsp;Express C library.

---

## Roadmap

- **NORA-B26 + Thread end-node** &mdash; *planned*. NORA-B26 is a u-blox
  Bluetooth&nbsp;LE 6.0 module based on Nordic's **nRF54L10** SoC, whose radio
  also supports **IEEE&nbsp;802.15.4**. A future release will add a
  single-SoC Matter-over-Thread end-node target on NORA-B26, mirroring the
  Wi-Fi end-node already shipped on NORA-W36. No source or binary is published
  yet &mdash; watch this repo's [Releases](https://github.com/u-blox/u-connectMatter/releases)
  page.
- **NORA-W36 firmware updates** &mdash; tracked against the
  [u-connectXpress NORA-W36 release notes](https://github.com/u-blox/u-connectXpress/tree/main/NORA-W36).

---

## License

Application code is released under the **Apache License 2.0**. The Matter SDK
(Connectivity Standards Alliance &ndash; Matter, formerly Project CHIP) is
distributed under its own Apache 2.0 license; see
[connectedhomeip](https://github.com/project-chip/connectedhomeip) for details.

---

## Support / contact

- u-blox support &mdash; <https://www.u-blox.com/en/support>
- Product page &mdash; <https://www.u-blox.com/en/product/nora-w36-series>
- u-connect&nbsp;Express &mdash; <https://github.com/u-blox/u-connectXpress>
- Matter specification &mdash; <https://csa-iot.org/all-solutions/matter/>
