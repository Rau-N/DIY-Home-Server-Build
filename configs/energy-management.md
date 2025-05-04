# 🔋 Energy Management for Proxmox Host

This guide describes system-level tweaks to reduce idle power consumption and improve energy efficiency on the Proxmox host. These changes include:

- CPU governor settings
- ASPM tuning
- PCI device autosuspend
- Optional SATA power management
- And verification using PowerTOP

---

## 📄 Script: `/root/powertop.sh`

Create a new script file at `/root/powertop.sh`:

```bash
vi /root/powertop.sh
```

Paste the following contents:

```bash
#!/bin/bash

# This command sets the CPU frequency scaling governor to "powersave" for all CPU cores.
# The "powersave" governor aims to run the CPU at the lowest possible frequency to reduce power consumption.
echo "powersave" | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# Set ASPM policy to powersave mode
echo powersave > /sys/module/pcie_aspm/parameters/policy

# Disable the kernel's NMI (Non-Maskable Interrupt) watchdog.
# This can free some CPU resources and potentially reduce power usage.
echo '0' > '/proc/sys/kernel/nmi_watchdog'

# Increase the dirty writeback interval to 15 seconds (1500 centiseconds).
# This delays how often the kernel flushes memory buffers to disk,
# reducing disk I/O and potentially saving power.
echo '1500' > '/proc/sys/vm/dirty_writeback_centisecs'

# Enable PCI device autosuspend
echo 'auto' > '/sys/bus/pci/devices/0000:00:00.0/power/control'
echo 'auto' > '/sys/bus/pci/devices/0000:01:00.0/power/control'
echo 'auto' > '/sys/bus/pci/devices/0000:02:00.0/power/control'
echo 'auto' > '/sys/bus/pci/devices/0000:00:08.0/power/control'
echo 'auto' > '/sys/bus/pci/devices/0000:00:14.2/power/control'
echo 'auto' > '/sys/bus/pci/devices/0000:00:16.3/power/control'
echo 'auto' > '/sys/bus/pci/devices/0000:00:17.0/power/control'
echo 'auto' > '/sys/bus/pci/devices/0000:00:1f.0/power/control'
echo 'auto' > '/sys/bus/pci/devices/0000:00:1f.5/power/control'
echo 'auto' > '/sys/bus/pci/devices/0000:03:00.0/power/control'

# SATA Power Management (disabled by default to retain hotplug support)
# for ((i = 0 ; i <= 7 ; i++ )); do
#     echo 'med_power_with_dipm' > "/sys/class/scsi_host/host$i/link_power_management_policy"
# done
# for ((i = 1 ; i <= 8 ; i++ )); do
#     echo 'auto' > "/sys/bus/pci/devices/0000:00:17.0/ata$i/power/control"
# done

exit 0
```

> 💡 You can uncomment the SATA section if you do **not** rely on hotplug functionality.


Make the script executable:

```bash
chmod +x /root/powertop.sh
```

---

## 🔁 Auto-Execution on Reboot

To apply this energy management script automatically at system startup, add it to `crontab`:

```bash
crontab -e
```

Append the following line:

```cron
@reboot /root/powertop.sh
```

This ensures the script runs every time the system boots.

---

## ✅ Optional: Use PowerTOP for Validation

[PowerTOP](https://wiki.ubuntuusers.de/PowerTOP/) is a tool to analyze and optimize power usage.

### Install PowerTOP

```bash
apt update
apt install powertop
```

### Run It

```bash
powertop
```

Navigate to the **"Tunables"** tab using the arrow keys. You want all entries to read:

```text
Good
```

If some show `Bad`, you can tune them live inside the interface or use:

```bash
powertop --auto-tune
```

> ⚠️ `--auto-tune` must be run manually or scripted at boot if you want its settings to persist.

---

## 🧪 Manual Checks

- Check CPU governor:

  ```bash
  cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
  ```

- Check ASPM policy:

  ```bash
  cat /sys/module/pcie_aspm/parameters/policy
  ```

---

## 🔗 Related

- [BIOS Power-Saving Configuration](../configs/bios-settings.md)

