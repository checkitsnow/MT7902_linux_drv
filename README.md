# MediaTek MT7902 Wi-Fi 6E Driver Installation Guide on Linux

Complete installation guide for **MediaTek MT7902** Wi-Fi 6E + Bluetooth drivers on Linux systems.

## Overview

MediaTek officially submitted an 11-patch patchset for MT7902 to the mainline Linux mailing list in February 2026. The driver is expected to land in Linux 7.1. Until then, use the out-of-tree driver by [hmtheboy154](https://github.com/hmtheboy154/mt7902), which supports kernels **6.6–6.19**.

⚠️ **Note:** Only the **PCIe version** of MT7902 is supported.

## Kernel Compatibility

Run this command to check your current kernel version:

```bash
uname -r
```

| Distribution | Kernel Version | Status |
|---|---|---|
| **Ubuntu 24.04 LTS / Kubuntu** | ~6.8 | ✅ Recommended |
| **Ubuntu 24.10 / Kubuntu** | ~6.11 | ✅ Works great |
| **Ubuntu 25.04 / Kubuntu** | ~6.14 | ✅ Works great |
| **Linux Mint 22** | ~6.8 | ✅ Works |
| **Arch Linux** | 6.x | ✅ Works |
| **Fedora 40/41** | 6.x | ✅ Works |
| **Ubuntu 22.04 LTS** | 5.15 | ❌ Kernel too old |

## Installation

### Step 0: Check Kernel Version

```bash
uname -r
```

### Part 1: Wi-Fi Driver Installation

#### 1. Install build dependencies

**Ubuntu / Debian / Linux Mint:**
```bash
sudo apt update
sudo apt install -y build-essential git linux-headers-$(uname -r) dkms
```

**Arch Linux:**
```bash
sudo pacman -S --needed base-devel git linux-headers dkms
```

**Fedora:**
```bash
sudo dnf install -y kernel-devel kernel-headers gcc make git dkms
```

#### 2. Clone the Wi-Fi driver repository

```bash
git clone https://github.com/hmtheboy154/mt7902 -b backport mt7902_wifi
cd mt7902_wifi
```

#### 3. Build and install the driver

```bash
sudo make install -j$(nproc)
```

#### 4. Install the required firmware

```bash
sudo make install_fw
```

#### 5. Reboot your system

```bash
sudo reboot
```

#### 6. Verify Wi-Fi is working

```bash
# Check network interfaces
ip link show

# Verify the module is loaded
lsmod | grep mt7902

# List available Wi-Fi networks
nmcli device wifi list
```

#### 7. If Wi-Fi isn't working try to copy mt7902e driver to mediatek dir:
```bash
sudo cp /lib/firmware/mediatek/mt7902e/* /lib/firmware/mediatek/

# update image of initramfs
sudo update-initramfs -u

# Reboot the driver module
sudo modprobe -r mt7902e
sudo modprobe mt7902e
# OR REBOOT PC
sudo reboot
```

### Part 2: Bluetooth Driver Installation

⚠️ **WARNING:** The Bluetooth driver is in a separate branch (`bluetooth_backport`). Install it independently from the Wi-Fi driver.

#### 1. Clone the Bluetooth driver repository

```bash
cd ~
git clone https://github.com/hmtheboy154/mt7902 -b bluetooth_backport btusb_mt7902
cd btusb_mt7902
```

#### 2. Build and install the driver + firmware

```bash
sudo make install -j$(nproc)
sudo make install_fw
```

#### 3. Blacklist conflicting kernel modules

The built-in `btusb` and `btmtk` modules conflict with `btusb_mt7902`. They must be blacklisted to prevent them from loading at boot.

```bash
sudo tee /etc/modprobe.d/blacklist_btusb.conf > /dev/null <<'EOF'
blacklist btusb
blacklist btmtk
EOF
```

#### 4. Reboot your system

```bash
sudo reboot
```

#### 5. Verify Bluetooth is working

```bash
# Show Bluetooth adapter info
bluetoothctl show

# List all wireless devices
rfkill list
```

## Troubleshooting

### Wi-Fi does not appear after reboot

**Check for conflicting modules:**

Blacklist the built-in mt7921e module:

```bash
sudo tee /etc/modprobe.d/blacklist-mt7921.conf > /dev/null <<'EOF'
blacklist mt7921e
blacklist mt7921_common
EOF
```

**Update initramfs (Ubuntu/Debian/Mint):**
```bash
sudo update-initramfs -u
```

**Update initramfs (Arch Linux):**
```bash
sudo mkinitcpio -P
```

**Try to reload the driver module:**
```bash
sudo modprobe mt7902e
```

### View firmware / driver errors in boot log

```bash
dmesg | grep -iE 'mt7902|mt76|firmware'
```

### Try copying driver to another directory

```bash
sudo cp /lib/firmware/mediatek/mt7902e/* /lib/firmware/mediatek/
```

### Uninstall Wi-Fi driver (rollback)

```bash
cd ~/mt7902_wifi
sudo make uninstall
sudo make uninstall_fw
```

### Uninstall Bluetooth driver (rollback)

```bash
cd ~/btusb_mt7902
sudo make uninstall
sudo make uninstall_fw
```

## Status Summary

| Component | Branch | Kernel Support | Status |
|---|---|---|---|
| **Wi-Fi (mt7902e)** | `backport` | 6.6–6.19 | ✅ Working |
| **Bluetooth** | `bluetooth_backport` | 6.6–6.19 | ✅ Working |
| **Mainline kernel** | — | Linux 7.1+ | 🔄 Coming soon |

## Resources

- **GitHub Repository:** [hmtheboy154/mt7902](https://github.com/hmtheboy154/mt7902)
- **Compatibility Spreadsheet:** [Google Sheets](https://docs.google.com/spreadsheets/d/1G2mQEeLQAu4oB85G-y4A9OduA1ZP0rUcY-b6MRnZhFU)
- **Support Discord:** [discord.gg/JGhjAxEFhz](https://discord.gg/JGhjAxEFhz)

## License

This guide is based on the official MT7902 installation guide. For driver source code and licensing, see the [hmtheboy154/mt7902](https://github.com/hmtheboy154/mt7902) repository.
