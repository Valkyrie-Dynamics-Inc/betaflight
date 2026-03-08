# Betaflight - Tactical Radar HUD Fork

## Overview
This is a specialized fork of Betaflight that integrates an external radar directly into the analog FPV video feed. 

It intercepts real-time tracking data via a custom MSP command and uses the onboard AT7456E OSD chip to project a dynamically scaling tactical bounding box with distance and speed telemetry over the target.

## Architecture Modifications
- **MSP Parser (`src/main/msp/msp.c` & `msp_protocol.h`):** Registered custom MSP command (ID `190`). The parser catches 8-byte payload packets on the configured UART and writes them to a global `radarData_t` struct.
- **OSD Renderer (`src/main/osd/osd_elements.c`):** Registered new OSD element (`OSD_RADAR_TARGET`). Bypasses standard static buffers to draw four bounding box corners dynamically and centers telemetry text above the target.

## Building from Source
Ensure you have the standard Betaflight build toolchain installed (e.g., Ubuntu/WSL).

1. Generate the target configurations:
   ```bash
   make configs
   ```

2. Compile the firmware for the Omnibus F4 SD target:
    ```bash
    make OMNIBUSF4SD
    ```

The compiled `.hex` file will be located in the `obj/` directory.

## Flashing & Configuration

### 1. Flash Firmware

Open Betaflight Configurator, go to the Firmware Flasher, load the compiled `obj/betaflight_OMNIBUSF4SD.hex` file locally, and flash the board. Click "Apply Custom Defaults" if prompted upon first connection.

### 2. Enable the UART Port

The FC needs to listen for the incoming radar serial data.

1. Go to the **Ports** tab.
2. Locate the UART connected to the radar (e.g., UART3).
3. Toggle **Configuration/MSP** to **ON**.
4. Set the Baud Rate to **115200**.
5. Click **Save and Reboot**.

### 3. Configure OSD Video Format

The AT7456E chip often fails to auto-detect the camera feed, resulting in a blank screen.

1. Go to the **OSD** tab.
2. Under Video Format, change it from `Auto` to **NTSC** (or PAL, depending on your exact camera specs).
3. Click **Save**.

### 4. Upload the Tactical Font

The default Betaflight font does not contain HUD targeting corners. We hijacked four unused ASCII slots (`180`, `181`, `182`, `183`) to act as 90-degree tactical brackets (⌜ ⌝ ⌞ ⌟).

1. Go to the **OSD** tab.
2. Click **Font Manager** (bottom right).
3. Click **Load Font** and select the custom `.mcm` file included in this repository `src/fonts/custom_fonts.mcm`.
4. Click **Upload Font** and wait for the upload to complete.
5. Ensure the `Radar Target` element is enabled in the OSD elements list.

## Current State & Next Steps

**Current (V1):** The pipeline successfully parses data and tracks a single dynamic target across the analog grid with proper camera FOV projection.

**Next Pipeline (V2):**

* **Swarm Tracking:** Convert the C-struct to an array to handle tracking up to 4 simultaneous targets.
* **Threat Assessment UI:** Implement dynamic UI logic (e.g., blink bounding box rapidly if target speed exceeds threshold or distance drops below a set radius, pointing arrow if the target is out of the FOV).



![Betaflight](https://raw.githubusercontent.com/betaflight/.github/main/profile/images/bf_logo.svg#gh-light-mode-only)
![Betaflight](https://raw.githubusercontent.com/betaflight/.github/main/profile/images/bf_logo_dark.svg#gh-dark-mode-only)

[![Latest version](https://img.shields.io/github/v/release/betaflight/betaflight)](https://github.com/betaflight/betaflight/releases) [![Build](https://img.shields.io/github/actions/workflow/status/betaflight/betaflight/push.yml?branch=master)](https://github.com/betaflight/betaflight/actions/workflows/push.yml) [![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0) [![Join us on Discord!](https://img.shields.io/discord/868013470023548938)](https://discord.gg/n4E6ak4u3c)

Betaflight is flight controller software (firmware) used to fly multi-rotor craft and fixed wing craft. Betaflight focuses on flight performance, leading-edge feature additions, and wide target support.


## Release Schedule

| Date  | Release | Stage | Status |
| - | - | - | - |
| 26-12-2025 | 2025.12 | Release | Completed |
| 01-04-2026 | 2026.6 | Beta | |
| 01-05-2026 | 2026.6 | Release Candidate | |
| 01-06-2026 | 2026.6 | Release | |
| 01-10-2026 | 2026.12 | Beta | |
| 01-11-2026 | 2026.12 | Release Candidate | |
| 01-12-2026 | 2026.12 | Release | |


## News

### 📣 Announcement: New Versioning Scheme & Release Cadence 📣

To create a more predictable release schedule, we're moving to a new versioning system and development cycle, starting with the next release.

**New Format**: `YYYY.M.PATCH` (e.g., `2025.12.1`)

**Release Cadence**: Two major releases per year.

**Target Months**: June and December.

This means the successor to our current `4.x` series will be Betaflight `2025.12.x`, followed by Betaflight `2026.6.x`. We will also align the Betaflight App and Firmware to the same `YYYY.M.PATCH` releases (and cadence).

**Our New Release Cycle**

To support this schedule, our development phases will be structured as follows:

**Alpha**: For new feature development. Alpha builds for the next version will be available shortly after a stable release is published.

**Beta**: A one-month feature freeze for bug fixes only, and existing pull requests currently being reviewed, starting approximately two months before a release.

**Release Candidate (RC)**: A one-month period for final stabilization and testing before the official release.

### Requirements for the submission of new and updated target configuration

The requirements for pull requests adding new targets or modifying existing targets are available on the [betaflight.com website](https://www.betaflight.com/docs/development/manufacturer/requirements-for-submission-of-targets).

## Features

Betaflight has the following features:

* Multi-color RGB LED strip support (each LED can be a different color using variable length WS2811 Addressable RGB strips - use for Orientation Indicators, Low Battery Warning, Flight Mode Status, Initialization Troubleshooting, etc)
* DShot (150, 300 and 600), Multishot, Oneshot (125 and 42) and Proshot1000 motor protocol support
* Blackbox flight recorder logging (to onboard flash or external microSD card where equipped)
* Support for targets that use the STM32 F4, G4, F7 and H7 processors
* PWM, PPM, SPI, and Serial (SBus, SumH, SumD, Spektrum 1024/2048, XBus, etc) RX connection with failsafe detection
* Multiple telemetry protocols (CRSF, FrSky, HoTT smart-port, MSP, etc)
* RSSI via ADC - Uses ADC to read PWM RSSI signals, tested with FrSky D4R-II, X8R, X4R-SB, & XSR
* OSD support & configuration without needing third-party OSD software/firmware/comm devices
* OLED Displays - Display information on: Battery voltage/current/mAh, profile, rate profile, mode, version, sensors, etc
* In-flight manual PID tuning and rate adjustment
* PID and filter tuning using sliders
* Rate profiles and in-flight selection of them
* Configurable serial ports for Serial RX, Telemetry, ESC telemetry, MSP, GPS, OSD, Sonar, etc - Use most devices on any port, softserial included
* VTX support for Unify Pro and IRC Tramp
* and MUCH, MUCH more.


## Installation & Documentation

See: https://betaflight.com/docs/wiki


## Support and Developers Channel

There's a dedicated [Discord server](https://discord.gg/n4E6ak4u3c) for help, support and general community.


## Betaflight Application

To configure Betaflight you should use the [Betaflight App](https://app.betaflight.com). It is a progressive web app, so should always be the latest version.


## Contributing

Contributions are welcome and encouraged. You can contribute in many ways:

* implement a new feature in the firmware or in the app (see [below](#Developers));
* documentation updates and corrections;
* How-To guides - received help? Help others!
* bug reporting & fixes;
* new feature ideas & suggestions;
* provide a new translation for the app, or help us maintain the existing ones (see [below](#Translators)).

The best place to start is the Betaflight Discord (registration [here](https://discord.gg/n4E6ak4u3c)). Next place is the github issue tracker:

https://github.com/betaflight/betaflight/issues
https://github.com/betaflight/betaflight-configurator/issues

Before creating new issues please check to see if there is an existing one, search first otherwise you waste people's time when they could be coding instead!

If you want to contribute to our efforts financially, please consider making a donation to us through [PayPal](https://paypal.me/betaflight).

If you want to contribute financially on an ongoing basis, you should consider becoming a patron for us on [Patreon](https://www.patreon.com/betaflight).


## Developers

Contribution of bugfixes and new features is encouraged. Please be aware that we have a thorough review process for pull requests, and be prepared to explain what you want to achieve with your pull request.
Before starting to write code, please read our [development guidelines](https://www.betaflight.com/docs/development) and [coding style definition](https://www.betaflight.com/docs/development/CodingStyle).

GitHub actions are used to run automatic builds


## Translators

We want to make Betaflight accessible for pilots who are not fluent in English, and for this reason we are currently maintaining translations into 21 languages for Betaflight Configurator: Català, Dansk, Deutsch, Español, Euskera, Français, Galego, Hrvatski, Bahasa Indonesia, Italiano, 日本語, 한국어, Latviešu, Português, Português Brasileiro, polski, Русский язык, Svenska, 简体中文, 繁體中文.
We have got a team of volunteer translators who do this work, but additional translators are always welcome to share the workload, and we are keen to add additional languages. If you would like to help us with translations, you have got the following options:
- if you help by suggesting some updates or improvements to translations in a language you are familiar with, head to [crowdin](https://crowdin.com/project/betaflight-configurator) and add your suggested translations there;
- if you would like to start working on the translation for a new language, or take on responsibility for proof-reading the translation for a language you are very familiar with, please head to the Betaflight Discord chat (registration [here](https://discord.gg/n4E6ak4u3c)), and join the ['translation'](https://discord.com/channels/868013470023548938/1057773726915100702) channel - the people in there can help you to get a new language added, or set you up as a proof reader.


## Hardware Issues

Betaflight does not manufacture or distribute their own hardware. While we are collaborating with and supported by a number of manufacturers, we do not do any kind of hardware support.

If you encounter any hardware issues with your flight controller or another component, please contact the manufacturer or supplier of your hardware, or check [Discord](https://discord.gg/n4E6ak4u3c) to see if others with the same problem have found a solution.


## Betaflight Releases

You can find our release [here](https://github.com/betaflight/betaflight/releases) on Github and we also have more detailed [release notes](https://www.betaflight.com/docs/category/release-notes) at [betaflight.com](https://www.betaflight.com).


## Open Source / Contributors

Betaflight is software that is **open source** and is available free of charge without warranty to all users.

For a complete list of contributors (past and present) see [Github](https://github.com/betaflight/betaflight/graphs/contributors).
