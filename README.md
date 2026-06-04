# MT7925 5GHz Injection Fix

This repository contains the patch and firmware for the MediaTek MT7925 (Wi-Fi 7) card to fix packet injection on the 5GHz band.

## The Issue
The `mt76` driver for MT7925 defaults to Band 0 (2.4GHz) when in monitor mode, failing to switch to Band 1 (5GHz/6GHz) for injected packets. This results in 100% packet loss during injection tests on 5GHz channels.

## The Fix
The included patch `mt7925_5ghz_injection.patch` adds a fallback to the driver's TX path. If the interface is in monitor mode and tuned to a 5GHz/6GHz frequency, it correctly assigns the hardware band index.

## Contents
- `mt7925_5ghz_injection.patch`: The surgical fix for `mt7925/mac.c`.
- `WIFI_MT7925_PATCH_MCU_1_1_hdr.bin`: MCU Patch Firmware.
- `WIFI_RAM_CODE_MT7925_1_1.bin`: RAM Code Firmware.
- `mt76_source/`: Pre-patched source code for the `mt76` driver.

## Installation
1.1 Install kernel headers: `sudo apt install raspberrypi-kernel-headers` for Raspberry Pi or any (arm64) devices,
1.2 Install kernel headers: `sudo apt install -y linux-headers-$(uname -r) build-essential bc dkms git libelf-dev rfkill iw` for (amd64).
2. Apply the patch to the `mt76` source.
3. Build and install:
   ```bash
   make -j$(nproc)
   sudo make install
   sudo modprobe -r mt7925e && sudo modprobe mt7925e
   ```
