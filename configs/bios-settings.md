# ⚙️ Custom BIOS Power-Saving Configuration

This document outlines the manual BIOS settings applied to the **CWWK Q670** motherboard using a custom BIOS to enable advanced **power-saving features**, such as **ASPM** and deep **C-states**. These settings help significantly reduce idle power consumption.

> 🛠️ These changes were applied via a custom BIOS, available in [`/bios`](../bios). Use at your own risk.

---

## ✅ Modified BIOS Settings

Navigate through your BIOS menu using keyboard arrows. The following options should be changed:

---

### 🔋 Power & Performance

**Path:**  
`Advanced > Power & Performance > CPU - Power Management Control`

| Setting                          | Value     |
|----------------------------------|-----------|
| C States                         | Enabled   |
| Package C State Limit            | C10       |

---

### 🧬 RC ACPI Settings

**Path:**  
`Advanced > RC ACPI Settings`

| Setting       | Value     |
|----------------|-----------|
| Native ASPM   | Enabled   |

---

### 🧩 Chipset > PCH-IO Configuration

**Path:**  
`Chipset > PCH-IO Configuration > PCI Express Configuration`

| Setting                          | Value                |
|----------------------------------|----------------------|
| PCH PCIE Power Gating            | Disabled             |
| PCI Express Root Port 1–28 > ASPM| L1                   |
| PCI Express Root Port 1–28 > L1 Substates | L1.1 and L1.2 |

---

### 🖥️ System Agent (SA) Configuration

**Path:**  
`Chipset > System Agent (SA) Configuration > PCI Express Configuration`

| Setting                          | Value     |
|----------------------------------|-----------|
| PCI Express Root Port 1–3 > ASPM | L1        |

---

## 🔍 Verifying ASPM Activation

To verify that ASPM (Active State Power Management) is properly enabled at the PCIe root ports and devices, run the following command:

```bash
lspci -vvPPDq | awk '/ASPM/{print $0}' RS= | grep --color -P '(^[a-z0-9:./]+|:\sASPM (\w+)? ?((En|Dis)abled)?)'
```

### ✅ Sample Output (from this system):

```text
0000:00:1a.0 PCI bridge: Intel Corporation Alder Lake-S PCH PCI Express Root Port #25 (rev 11) (prog-if 00 [Normal decode])
                LnkCtl: ASPM L1 Enabled; RCB 64 bytes, Disabled- CommClk+
0000:00:1b.0 PCI bridge: Intel Corporation Alder Lake-S PCH PCI Express Root Port (rev 11) (prog-if 00 [Normal decode])
                LnkCtl: ASPM L1 Enabled; RCB 64 bytes, Disabled- CommClk+
0000:00:1c.0 PCI bridge: Intel Corporation Alder Lake-S PCH PCI Express Root Port (rev 11) (prog-if 00 [Normal decode])
                LnkCtl: ASPM L1 Enabled; RCB 64 bytes, Disabled- CommClk+
0000:00:1c.3 PCI bridge: Intel Corporation Device 7abb (rev 11) (prog-if 00 [Normal decode])
                LnkCtl: ASPM L1 Enabled; RCB 64 bytes, Disabled- CommClk+
0000:00:1a.0/01:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller S4LV008[Pascal] (prog-if 02 [NVM Express])
                LnkCtl: ASPM L1 Enabled; RCB 64 bytes, Disabled- CommClk+
0000:00:1b.0/02:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983 (prog-if 02 [NVM Express])
                LnkCtl: ASPM L1 Enabled; RCB 64 bytes, Disabled- CommClk+
0000:00:1c.0/03:00.0 Ethernet controller: Intel Corporation Ethernet Controller I226-LM (rev 04)
                LnkCtl: ASPM L1 Enabled; RCB 64 bytes, Disabled- CommClk+
0000:00:1c.3/04:00.0 Ethernet controller: Intel Corporation Ethernet Controller I226-V (rev 04)
                LnkCtl: ASPM L1 Enabled; RCB 64 bytes, Disabled- CommClk+
```

If you see `ASPM L1 Enabled` across all relevant devices, ASPM is successfully active.

---

## 💡 Notes

- These settings are required to fully activate **PCIe power saving (ASPM)** and deep sleep states.
- Default BIOS versions often **hide or disable** these features — the custom BIOS exposes and enables them.

## 🔗 Related

- [BIOS flashing instructions](../bios/README.md)
- [Energy Management for Proxmox Host](../configs/energy-management.md)
- [ZFS + IOMMU Setup Guide](../docs/zfs-setup.md)