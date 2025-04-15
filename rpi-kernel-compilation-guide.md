# Compiling and Installing Custom Raspberry Pi Kernel

This guide explains how to compile the Raspberry Pi kernel for 64-bit systems (e.g., Raspberry Pi 4/5), install kernel modules, and deploy the compiled kernel to your Raspberry Pi.

---

## 🧰 Prerequisites

Your Raspberry Pi Board is running any latest official Raspberry Pi OS.
Confirm the running kernel version with:

```bash
uname -a
```

Install the cross-compilation toolchain on your host (Ubuntu):

```bash
sudo apt install crossbuild-essential-arm64
```

---

## 📥 Clone the Raspberry Pi Linux Kernel Source

```bash
git clone --depth=1 --branch rpi-6.6.y https://github.com/raspberrypi/linux
cd linux
```

---

## ⚙️ Kernel Configuration and Compilation

Set environment variable:

```bash
# this is required because the raspberry-pi firmware understands this name only
KERNEL=kernel8
```

### 1. Load default config:
I have Raspberry Pi 4B so, I have selected bcm2711_defconfig

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
```

### 2. (Optional) Customize config:
I needed it to enable couple of device drivers, so I did.

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

### 3. Build the kernel, modules, and device tree blobs:
```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs
```

---

## 📦 Install Kernel Modules

Install modules to a mount path (replace `mnt/rpi-kernel` with your actual RPi rootfs mount point):

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=mnt/rpi-kernel modules_install
```

At this point you have the kernel images ready with you targeted for your Raspberry Pi board.

---

## 🔐 SSH Access to Raspberry Pi

1. Login to the **serial console** or directly on the device.
2. Configure WiFi using:
   ```bash
   sudo raspi-config
   ```
3. Enable the SSH service:
   ```bash
   sudo systemctl enable ssh
   ```
4. Allow root SSH login by editing `/etc/ssh/sshd_config`:
   ```bash
   PermitRootLogin yes
   ```
5. Reboot the device:
   ```bash
   sudo reboot
   ```

---

## 📁 Deploy the Compiled Kernel to Raspberry Pi

Assuming the SD card or Pi rootfs is mounted under `mnt/`:

Copy kernel and DTB images:
```bash
sudo cp mnt/boot/$KERNEL.img mnt/boot/$KERNEL-backup.img
sudo cp arch/arm64/boot/Image mnt/boot/firmware/$KERNEL.img
sudo cp -R arch/arm64/boot/dts/broadcom/*.dtb mnt/boot/firmware/
sudo cp -R arch/arm64/boot/dts/overlays/*.dtb* mnt/boot/firmware/overlays/
sudo cp arch/arm64/boot/dts/overlays/README mnt/boot/firmware/overlays/
```

Copy Kernel Modules:
All the kernel modules are kept in your kernel source directory `mnt/rpi-kernel/lib`
Copy the `mnt/rpi-kernel/lib/modules/6.6.78-v8+` to RPi rootfs `mnt/lib/` 
```bash
sudo cp -R mnt/rpi-kernel/lib/modules/6.6.78-v8+ mnt/lib/modules/
```

---

## ✅ Done!

Your Raspberry Pi should now be running the newly compiled kernel after a reboot. You can confirm the kernel version with:

```bash
uname -a
```
