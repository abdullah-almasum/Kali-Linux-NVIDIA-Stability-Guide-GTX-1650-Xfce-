🛠️ Kali Linux NVIDIA GUI Troubleshooting
This repository provides a comprehensive fix for GUI freezing, "sticking," and sync issues on Kali Linux when using NVIDIA GTX 1650 hardware in a dual-monitor Xfce environment.  

📂 Repository Contents
Nvidia_Troubleshooting_GUI_Stuck.m: An automated script to handle the driver installation and configuration sequence.  

Nvidia_Troubleshooting_GUI_Stuck_Clean_Version.md: A detailed, step-by-step manual guide covering the "Clean Slate" process and manual X11 configuration.  

🚀 Quick Overview
If your Kali Linux desktop is freezing on a dual-monitor setup, it is likely due to a sync bottleneck between the open-source drivers and the kernel. This project helps you:  

Purge conflicting Nouveau and old NVIDIA drivers.  

Install the proprietary NVIDIA driver with DKMS support for kernel stability.  

Enable PRIME Synchronization to fix dual-monitor lag.  

Optimize Xfce by disabling the compositor and forcing the NVIDIA Pipeline.  

✅ How to Verify
After applying the fixes and rebooting, run the following in your terminal:

Bash
nvidia-smi
If your GTX 1650 is listed and showing memory usage for Xorg, the setup was successful.  

⚠️ Maintenance
Because Kali is a rolling release, kernel updates may occasionally break the driver. If your GUI "sticks" again after an update, simply run:  

Bash
sudo apt install --reinstall nvidia-kernel-dkms
