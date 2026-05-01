## 🛠️ Kali Linux NVIDIA Stability Guide (GTX 1650 & Xfce)

This guide addresses GUI freezing and "sticking" issues caused by driver conflicts, dual-monitor sync errors, and kernel mismatches.

### Phase 1: The "Clean Slate"
Before installing new drivers, you must remove "ghost" drivers and the open-source Nouveau driver which conflict with proprietary hardware.

1.  **Purge existing drivers:**
    ```bash
    sudo apt purge nvidia* -y
    sudo apt autoremove -y
    sudo rm /etc/X11/xorg.conf
    ```
2.  **Blacklist the Nouveau driver:**
    ```bash
    sudo bash -c "echo -e 'blacklist nouveau\noptions nouveau modeset=0' > /etc/modprobe.d/blacklist-nouveau.conf"
    sudo update-initramfs -u
    ```

---

### Phase 2: Driver Installation & Kernel Hooking
For a desktop i5/GTX 1650 setup, we use **DKMS** (Dynamic Kernel Module Support). This ensures that when Kali updates its kernel, your NVIDIA driver rebuilds itself automatically.

1.  **Install the Meta-Package and Headers:**
    ```bash
    sudo apt update
    sudo apt install -y linux-headers-$(uname -r) build-essential nvidia-driver nvidia-kernel-dkms nvidia-xconfig nvidia-settings
    ```
2.  **Identify your GPU BusID:**
    ```bash
    lspci | grep -i nvidia
    ```
    *Note: For a GTX 1650, this is usually `01:00.0` (formatted as `PCI:1:0:0` in config files).*

---

### Phase 3: Hardware Synchronization (Dual-Monitor Fix)
Dual-monitor setups often freeze because the two displays fall out of sync. **PRIME Synchronization** is the critical fix here.

1.  **Enable Modesetting in GRUB:**
    *   Backup: `sudo cp /etc/default/grub /etc/default/grub.bak`
    *   Edit: `sudo nano /etc/default/grub`
    *   Find `GRUB_CMDLINE_LINUX_DEFAULT` and change it to:
        > `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvidia-drm.modeset=1"`
2.  **Update GRUB:**
    ```bash
    sudo update-grub
    ```

---

### Phase 4: Manual X11 Configuration
If `nvidia-xconfig` fails to query the GPU, you must manually point the OS to the hardware.

1.  **Generate/Edit Xorg Config:**
    ```bash
    sudo nvidia-xconfig
    sudo nano /etc/X11/xorg.conf
    ```
2.  **Append/Verify these blocks at the bottom of the file:**
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
    EndSection
    ```

---

### Phase 5: Xfce UI Optimization
Xfce’s default compositor can stutter when paired with NVIDIA’s own composition pipeline.

1.  **Disable Xfce Compositor:**
    *   Go to **Settings > Window Manager Tweaks > Compositor**.
    *   Uncheck **"Enable display compositing"**.
2.  **Enable NVIDIA Full Composition Pipeline:**
    *   Open `nvidia-settings`.
    *   Go to **X Server Display Configuration > Advanced**.
    *   Check **Force Full Composition Pipeline** for **both** monitors.
    *   Click **Apply** and **Save to X Configuration File**.



---

### ⚠️ Maintenance & Troubleshooting

**The "Safe" Update Command:**
Since Kali is a rolling release, use this string to ensure everything stays in sync:
```bash
sudo apt update && sudo apt -y full-upgrade && sudo apt autoremove -y && sudo reboot
```

**If the GUI freezes after a Kernel update:**
The DKMS might have failed to hook. Reinstall it:
```bash
sudo apt install --reinstall nvidia-kernel-dkms
```

**Verification:**
Run `nvidia-smi`. If you see your **GTX 1650** listed with memory being used by `/usr/lib/xorg/Xorg`, your GPU is now successfully doing the heavy lifting.
