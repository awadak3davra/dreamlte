# Enhanced Kernel — Samsung Galaxy S8 / Note8 (Exynos 8895)

A fork of [exynos-linux-stable/dreamlte](https://github.com/exynos-linux-stable/dreamlte)
(Linux **4.4.194**, branch `tw90-android`) with backported drivers, filesystems and the fixes
needed to actually build with a modern toolchain and boot on real hardware.

Target SoC: **Samsung Exynos 8895** (`universal8895` / `dreamlte`, arm64).

---

## What's new in this fork

### 🩹 Boot fix (critical)
- **ASoC ABOX (audio DSP):** restore `memcpy_toio()` (instead of a plain `memcpy()`) for the
  ABOX SRAM IPC path and firmware upload. On arm64 a plain `memcpy()` into the ioremap'd
  device-memory region issues unaligned `ldp/stp` accesses, which fault — the kernel aborts
  early in boot. The stock tree does **not** boot on hardware without this; with it, it does.

### 🌐 Networking
- **WireGuard** VPN, in-tree (`net/wireguard`).
- **Realtek r8152 vendor driver** — RTL8152 / 8153 / 8155 / **8156**, adds **2.5 GbE** USB-Ethernet.
- Extra USB-NIC drivers: AX8817X, SMSC75xx / 95xx, RTL8150, CDC-NCM / EEM / MBIM, NET1080.
- **fq_codel** / **fq** queueing disciplines.
- **NFS server** (nfsd v4) and **CIFS** client.

### 💾 Filesystems
- **Mainline exFAT** driver (`fs/exfat`).
- **Btrfs** and **F2FS** enabled.
- **NTFS** (read) and **zstd** compression (`CRYPTO_ZSTD`).

### 🔌 Storage / USB
- **UAS** (USB Attached SCSI) for faster USB mass storage.

### 🎮 Input
- Gamepads / joysticks: `INPUT_JOYDEV`, Xbox (`xpad`), Sony DualShock (`hid-sony`), and more.

### 🧠 Memory
- **KSM** (Kernel Samepage Merging).

### 🔧 Build fixes
Lets the 4.4.194 base build with an aarch64 GCC 8.3 cross-toolchain on a modern (GCC 10+) host:
- `-fcommon` for the host tools (GCC 10+ defaults to `-fno-common`).
- Remove leftover git merge-conflict markers in `kernel/cred.c`.
- Drop a duplicate `binder_transaction_data_secctx` definition.
- Rename an f2fs-local `__blkdev_issue_discard` to avoid clashing with the block layer.

---

## Building

```sh
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-        # ARM GCC 8.3 (2019.03) recommended

make exynos8895-dreamlte-enhanced_defconfig    # config with all of the above enabled
make -j"$(nproc)" Image
```

Output: `arch/arm64/boot/Image`. Pack it into a boot image with `magiskboot` (keep the
device's original ramdisk + DT) or your usual AnyKernel/repack flow, then flash the `BOOT`
partition. Keep a backup of the original boot image for rollback.

---

## Credits

- Base tree: **exynos-linux-stable / dreamlte** (Linux 4.4.194).
- exFAT: upstream mainline driver.
- r8152 2.5G: Realtek vendor driver.
- WireGuard: Jason A. Donenfeld / wireguard-linux-compat.
