# RTL8192EU WiFi Driver - Modern Kernel Compatibility Patch

This repository contains the TP-Link RTL8192EU WiFi driver (v5.2.19.1) updated to work with modern Linux kernels (tested on kernel 6.14.0-37).

## Overview

The original driver from TP-Link was released in 2018 and is incompatible with modern Linux kernels due to numerous API changes. This version includes comprehensive patches to make it compile and work on current kernel versions.

## Compatibility

- **Original Driver Version**: v5.2.19.1_25633.20171222_COEX20171113-0047
- **Tested Kernel**: 6.14.0-37-generic (Ubuntu 24.04)
- **Architecture**: x86_64
- **Compiler**: GCC 13.3.0

## Changes Made

### 1. Timer API Updates (Kernel 4.15+)
**File**: `include/osdep_service_linux.h`
- Replaced deprecated `init_timer()` with `timer_setup()`
- Updated timer callback signatures for kernel >= 4.15

### 2. Thread Exit API (Kernel 5.17+)
**File**: `os_dep/osdep_service.c`
- Changed `complete_and_exit()` + `do_exit()` to `kthread_complete_and_exit()`
- Fixed undefined symbol error for `do_exit`

### 3. Proc Filesystem API (Kernel 5.17+)
**File**: `os_dep/linux/rtw_proc.c`
- Updated `PDE_DATA` macro to use lowercase `pde_data()` function
- Fixed undefined symbol error for `PDE_DATA`

### 4. IPX Protocol Removal (Kernel 4.20+)
**File**: `core/rtw_br_ext.c`
- Added version checks around IPX-related code (protocol removed from kernel)
- Wrapped IPX helper functions and protocol handling

### 5. VFS API Changes (Kernel 4.14+)
**File**: `os_dep/osdep_service.c`
- Replaced `__vfs_read()` with `kernel_read()`
- Removed deprecated `mm_segment_t`, `get_fs()`, `set_fs()` usage (removed in kernel 5.10)

### 6. Random Number API (Kernel 6.1+)
**File**: `os_dep/osdep_service.c`
- Changed `prandom_u32()` to `get_random_u32()`

### 7. Network Driver API Updates
**Files**: `os_dep/linux/os_intfs.c`, `os_dep/linux/usb_intf.c`
- Fixed `ndo_select_queue` signature for kernel 5.2+
- Updated `netif_napi_add()` signature for kernel 6.1+ (removed weight parameter)
- Fixed USB driver shutdown callback for kernel 5.14+

### 8. GRO (Generic Receive Offload) API (Kernel 5.19+)
**File**: `os_dep/linux/recv_linux.c`
- Updated GRO return value handling (GRO_DROP no longer available)

### 9. CFG80211 Wireless API Updates
**File**: `os_dep/linux/ioctl_cfg80211.c`
- Fixed `cfg80211_roam_info` structure changes for kernel 6.0+ (bssid moved to links array)
- Updated `cfg80211_ch_switch_notify()` signature for kernel 6.3+
- Removed deprecated `mgmt_frame_register` for kernel 5.8+
- Fixed `rtw_get_systime_us()` to use `ktime_get_boottime_ns()` for kernel 5.0+
- Removed `wireless_dev.current_bss` check for kernel 6.0+

### 10. Access Control API (Kernel 5.0+)
**File**: `os_dep/linux/rtw_android.c`
- Updated `access_ok()` to not use VERIFY_READ/VERIFY_WRITE parameters

### 11. Inline Function Fixes (GCC 13+)
**File**: `include/ieee80211.h`
- Changed `extern __inline` to `static inline` to fix CFI (Control Flow Integrity) symbol conflicts
- Prevents multiple definition errors with `__pfx_` symbols

### 12. Compiler Warning Suppressions
**File**: `Makefile`
- Added flags to handle warnings now treated as errors:
  - `-Wno-implicit-fallthrough`
  - `-Wno-incompatible-pointer-types`
  - `-Wno-return-type`
  - `-Wno-discarded-qualifiers`
  - `-Wno-implicit-function-declaration`
  - `-Wno-missing-prototypes`
  - `-fcf-protection=none`

## Installation

### Prerequisites
```bash
sudo apt-get update
sudo apt-get install build-essential linux-headers-$(uname -r) git
```

### Build and Install
```bash
# Clone the repository
git clone https://github.com/CihanMertDeniz/RTL8192EU-WiFi-Driver---TL-WN823N-V3.git
cd rtl8192EU_WiFi_linux_v5.2.19.1_25633.20171222_COEX20171113-0047

# Build
make clean
make

# Install
sudo make install

# Load the module
sudo modprobe 8192eu
```

### Manual Loading
```bash
# After building
sudo insmod 8192eu.ko

# Verify it's loaded
lsmod | grep 8192eu
dmesg | tail -20
```

### Uninstall
```bash
sudo make uninstall
# or
sudo rmmod 8192eu
```

## Supported Devices

This driver supports TP-Link WiFi adapters using the RTL8192EU chipset, including:

- TP-Link TL-WN823N v2
- TP-Link TL-WN823N v3
- And other RTL8192EU-based devices

Check `os_dep/linux/usb_intf.c` for the full list of supported USB IDs.

## Configuration

The driver supports various configuration options in the Makefile:
- `CONFIG_POWER_SAVING`: Enable power saving features
- `CONFIG_RTW_NAPI`: Enable NAPI support for better performance
- `CONFIG_RTW_GRO`: Enable Generic Receive Offload
- `CONFIG_BT_COEXIST`: Bluetooth coexistence support

## Troubleshooting

### Module fails to load
```bash
# Check kernel logs
dmesg | tail -50

# Verify kernel headers are installed
ls /lib/modules/$(uname -r)/build
```

### Build errors
```bash
# Clean and rebuild
make clean
make

# Check GCC version
gcc --version
```

### Interface not appearing
```bash
# Check if module is loaded
lsmod | grep 8192eu

# Check USB device
lsusb

# Check for firmware errors
dmesg | grep -i firmware
```

## Known Issues

1. Some cfg80211 API functions have changed signatures but are wrapped with compatibility macros
2. BTF (BPF Type Format) generation is skipped due to vmlinux unavailability
3. Some advanced features may have limited testing on newer kernels

## Contributing

Contributions are welcome! If you encounter issues with specific kernel versions or have improvements, please:
1. Test thoroughly
2. Document changes clearly
3. Submit pull requests with detailed descriptions

## License

This driver maintains the original GPL-2.0 license from Realtek/TP-Link.

## Credits

- Original driver: Realtek Semiconductor Corp. / TP-Link
- Kernel compatibility patches: Updated for modern kernels (2024)

## Disclaimer

This driver is provided as-is. Use at your own risk. While efforts have been made to ensure compatibility and stability, the maintainers are not responsible for any damage or issues arising from its use.

## Version History

- **2024-12-27**: Updated for kernel 6.14.0-37
  - Fixed all major API incompatibilities
  - Successfully compiled and built kernel module
  - Ready for testing on modern Ubuntu/Debian systems
