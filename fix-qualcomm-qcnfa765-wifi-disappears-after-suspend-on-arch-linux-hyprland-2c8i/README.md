## Intro

If you're running **Arch Linux (especially with Hyprland)** and your WiFi works after boot but disappears after locking or suspending, this post is for you.

This issue affects systems using:

- **Qualcomm QCNFA765**
- Kernel driver: `ath11k_pci`
- NetworkManager + wpa_supplicant
- GRUB managed by another distro (e.g., Ubuntu in a triple boot setup)

---

## The Problem

WiFi works normally after reboot.

But after:

- Locking the system
- Suspending
- Disconnecting and reconnecting

You get:

- `nmcli device wifi connect <ssid>` --ask → **timeout**
- `nmcli device` → no WiFi device listed
- `ip link` → no `wlan0` or `wlp*`
- `rfkill list` → not blocked

The WiFi interface disappears completely until reboot.

---

## Diagnosis

Check your WiFi chipset:

```bash
lspci -k | grep -A4 -Ei 'network|wireless'
```

If you see:

```bash
Qualcomm Technologies QCNFA765
Kernel driver in use: ath11k_pci
```

You're hitting a known power management bug.

---

## Root Cause

The `ath11k_pci` driver sometimes:

- Enters a deep idle power state (D3cold)
- Fails to resume properly after suspend
- The kernel drops the WiFi interface
- NetworkManager can no longer see a device

This is a resume power-state issue with ath11k chipsets.

---

## Temporary Fix (No Reboot)

Reloading the driver restores WiFi:

```bash
sudo modprobe -r ath11k_pci
sudo modprobe ath11k_pci
```

After that:

```bash
ip link
```

Your WiFi interface should return.

This confirms the issue is driver-related.

---

## Permanent Fix

### Disable ath11k Idle Power Saving

Add this kernel parameter:

```bash
ath11k_pci.disable_idle_ps=1
```

This prevents the driver from entering the unstable idle state.

---

### If you use GRUB

Edit:

```bash
sudo nano /etc/default/grub
```

Find:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="..."
```

Append:

```bash
ath11k_pci.disable_idle_ps=1
```

Example:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash ath11k_pci.disable_idle_ps=1"
```

Regenerate GRUB:

```bash
sudo update-grub
```

Verify:

```bash
cat /proc/cmdline
```

You should see:

```bash
ath11k_pci.disable_idle_ps=1
```

Done.

---

### Alternative: Auto-Reload Driver on Resume (Arch Only)

If you prefer not to modify GRUB, create a resume hook.

Create:

```bash
sudo nano /usr/lib/systemd/system-sleep/ath11k-resume
```

Add:

```bash
#!/bin/sh
case "$1" in
  post)
    /usr/bin/systemctl restart NetworkManager
    /usr/bin/modprobe -r ath11k_pci 2>/dev/null
    /usr/bin/modprobe ath11k_pci
    /usr/bin/systemctl restart NetworkManager
    ;;
esac
```

Make executable:

```bash
sudo chmod +x /usr/lib/systemd/system-sleep/ath11k-resume
```

Now WiFi will recover automatically after suspend.

---

## Summary

| Issue                          | Cause                       | Fix                       |
| ------------------------------ | --------------------------- | ------------------------- |
| WiFi disappears after suspend  | ath11k idle power-state bug | Disable idle power saving |
| `ip link` shows no wlan device | Driver failed resume        | Reload `ath11k_pci`       |
| nmcli timeout                  | No kernel interface         | Fix power state           |

---

## Final Thoughts

This issue can be confusing because:

- NetworkManager looks broken
- `nmcli` times out
- No device appears
- Reboot "magically" fixes it

But the real culprit is `ath11k` power management during suspend.

If you're running Hyprland on Arch with Qualcomm WiFi and experiencing this, hopefully this saves you a few hours.

Happy hacking 🐧# Intro
