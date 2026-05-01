# Kali Linux GUI Freezing: Causes and Fixes

With a GTX 1650 and a dual-monitor setup on Xfce, your system is likely hitting a "sync" bottleneck. The standard open-source drivers often struggle to manage two displays on NVIDIA hardware, leading to the GUI "stuck" or "hanging" sensation.

## 1. The "Ghost" Driver Cleanout
Sometimes multiple driver versions (nouveau and nvidia) try to load simultaneously, causing the GUI to "stick" as they fight for control of the GPU.
Run this to purge everything and start fresh:
```bash
sudo apt purge nvidia*
sudo apt autoremove
sudo apt update
```

## 2. Kill the "Xfce Compositor"
Xfce’s built-in compositor handles window transparency and shadows, but it is a frequent cause of GUI hangs when using two monitors with NVIDIA.

1. Go to **Settings > Window Manager Tweaks**.
2. Select the **Compositor** tab.
3. Uncheck **"Enable display compositing."**

**Note:** Your windows will lose their shadows, but the GUI will be significantly more stable.

---

## 3. Install the Correct Proprietary Drivers
Kali’s default drivers (nouveau) are notorious for freezing on dual-monitor NVIDIA setups. You need the official non-free drivers to handle the load.

### Update and Install:
```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y linux-headers-$(uname -r) build-essential nvidia-driver nvidia-xconfig nvidia-settings
```

### Generate Config:
Run this to let the system know you have an NVIDIA card as your primary:
```bash
sudo nvidia-xconfig
```

**Reboot:** You must reboot for these kernel changes to take effect.

---

## 4. Force-Install the Correct 2026 Headless/Desktop Stack
For a desktop i5/GTX 1650 setup, you need the specific DKMS (Dynamic Kernel Module Support) package so the driver re-builds correctly when the kernel updates.

```bash
sudo apt install -y nvidia-driver nvidia-dkms-desktop nvidia-settings nvidia-xconfig
```

### Identify and Configure the GPU
Because you have a desktop with an i5 (which likely has integrated graphics) and a GTX 1650, you need to tell Kali to use the NVIDIA card for the Xfce display.

1. **Find your Bus ID:**
   ```bash
   nvidia-xconfig --query-gpu-info | grep 'BusID'
   ```
   It will likely look like `PCI:1:0:0`.

2. **Generate the Configuration:**
   ```bash
   sudo nvidia-xconfig
   ```
   This forces the creation of a new `/etc/X11/xorg.conf` file that explicitly tells the OS to use the NVIDIA chip for both displays.

---

## 5. Critical: Enable "PRIME Synchronization"
Dual-monitor freezing on NVIDIA is often caused by the displays falling out of sync. You must enable PRIME Sync via the kernel.

1. **Backup Grub:** `sudo cp /etc/default/grub /etc/default/grub.bak`
2. **Open Grub config:** `sudo nano /etc/default/grub`
3. **Modify line:** Find the line starting with `GRUB_CMDLINE_LINUX_DEFAULT`.
4. **Add parameter:** Add `nvidia-drm.modeset=1` inside the quotes. It should look like this:
   `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvidia-drm.modeset=1"`
5. **Save and Exit:** (Ctrl+O, Enter, Ctrl+X).
6. **Apply changes and reboot:**
   ```bash
   sudo update-grub
   sudo reboot
   ```

### How to verify it worked?
After rebooting, open your terminal and type:
```bash
nvidia-smi
```
If you see a table showing your GTX 1650 and its memory usage, the driver is active.

---

## Troubleshooting: Manual "Hard Reset" Plan
If `nvidia-xconfig` fails or the GPU isn't recognized, follow these phases.

### Phase 1: The Clean Slate
```bash
sudo rm /etc/X11/xorg.conf
sudo apt purge nvidia* -y && sudo apt autoremove -y
```

### Phase 2: The Manual "Bypass" Install
1. **Install headers and driver:**
   ```bash
   sudo apt update
   sudo apt install -y linux-headers-$(uname -r) nvidia-driver nvidia-cuda-toolkit
   ```
2. **Manually identify your GPU:**
   ```bash
   lspci | grep -i nvidia
   ```
   Example output: `01:00.0 VGA compatible controller...` (BusID is `PCI:1:0:0`).

### Phase 3: Force-Load Installation
```bash
sudo apt update
sudo apt install -y nvidia-driver nvidia-dkms-desktop xserver-xorg-video-nvidia
```
*If not found, use:*
```bash
sudo apt install -y nvidia-driver nvidia-kernel-dkms
```

---

## If "nvidia-xconfig" is Missing

1. **Install utility:**
   ```bash
   sudo apt install -y nvidia-xconfig
   ```
2. **Generate Config:**
   ```bash
   sudo nvidia-xconfig
   ```
3. **Manual Xorg Update:**
   Backup: `sudo cp /etc/X11/xorg.conf /etc/X11/xorg.conf.bak`
   Edit: `sudo nano /etc/X11/xorg.conf`
   
   Add these blocks at the **bottom** of the file:
   ```text
   Section "Device"
       Identifier     "Device0"
       Driver         "nvidia"
       VendorName     "NVIDIA Corporation"
       BoardName      "GeForce GTX 1650"
       BusID          "PCI:1:0:0"
   EndSection

   Section "Screen"
       Identifier     "Screen0"
       Device         "Device0"
       Monitor        "Monitor0"
       DefaultDepth    24
       Option         "AllowEmptyInitialConfiguration" "True"
       SubSection     "Display"
           Depth       24
       EndSubSection
   EndSection
   ```

---

## 6. Fix "Screen Tearing" Freeze
1. Open NVIDIA Settings: `nvidia-settings`
2. Go to **X Server Display Configuration**.
3. Click **Advanced...** at the bottom.
4. Check **Force Full Composition Pipeline** for both monitors.
5. Click **Apply**, then **Save to X Configuration File**.

---

## ⚠️ Maintenance Note
After a `full-upgrade`, if the GUI is stuck again, the driver might need a rebuild:
```bash
sudo apt install --reinstall nvidia-kernel-dkms
```

**Recommended Safest One-Liner for Updates:**
```bash
sudo apt update && sudo apt -y full-upgrade && sudo apt autoremove -y && sudo apt autoclean -y && sudo reboot
```
