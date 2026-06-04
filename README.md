# MT7925 5GHz Injection Fix

This repository contains a patched MediaTek `mt76` driver to enable packet injection on the 5GHz band for the MT7925 (Filogic 360) chipset.

## The Issue
The original `mt76` driver incorrectly defaults to Band 0 (2.4GHz) when in monitor mode, even when tuned to 5GHz frequencies. This patch adds a mandatory fallback to detect the current band/frequency and set the correct hardware pipeline (TGID) for packet transmission.

## Requirements
- Kernel version 6.7+ (Recommended: 7.0+)
- Kernel headers for your specific kernel version (`linux-headers-$(uname -r)`)
- `build-essential`, `bc`, `git`, `iw`

## Installation Procedure

1. **Prerequisites & Cleanup**
   ```bash
   sudo apt update
   sudo apt install -y build-essential linux-headers-$(uname -r) bc git iw
   # Kill conflicting services
   sudo airmon-ng check kill
   # Remove old driver files
   sudo find /lib/modules/$(uname -r)/kernel/drivers/net/wireless/mediatek/mt76/ -name "mt7925*.ko" -exec rm -f {} \;
   ```

2. **Clone & Compile**
   ```bash
   git clone https://github.com/Ashcal9669/MT7925.git
   cd MT7925/mt76_source
   # Build the modules
   make -C /lib/modules/$(uname -r)/build M=$PWD modules
   ```

3. **Install & Load**
   ```bash
   # Install modules
   sudo make -C /lib/modules/$(uname -r)/build M=$PWD modules_install
   sudo depmod -a
   # Decompress modules (if your system stores them as .ko.xz)
   cd /lib/modules/$(uname -r)/updates/
   sudo unxz mt76.ko.xz mt76-connac-lib.ko.xz mt792x-lib.ko.xz
   sudo unxz mt7925/*.ko.xz
   sudo depmod -a
   # Load the driver
   sudo modprobe mt7925e
   ```

4. **Verify & Use**
   To enable injection on 5GHz, you **must** set the regulatory domain:
   ```bash
   # Set to a domain that allows 5GHz transmission (e.g., US)
   sudo iw reg set US
   # Set channel and test
   sudo iw dev wlp6s0 set type monitor
   sudo iw dev wlp6s0 set channel 157
   sudo aireplay-ng --test wlp6s0
   ```
