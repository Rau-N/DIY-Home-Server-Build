# 🧬 Custom BIOS for CWWK Q670

This folder contains a custom BIOS archive to unlock advanced power-saving options such as ASPM and C-states on the CWWK Q670 motherboard.

---

## 📦 Contents

- `BIOS Q670-PLUS unlocked v7.rar`  
  After extraction, the USB root should contain:

```
/
├── CW-Q670-PLUS/
├── EFI/
└── auto.nsh
```

The flashing process is automated via a UEFI shell script (`auto.nsh`).

---

## 🪛 Step 1: Prepare a FAT32 USB Stick

1. Use a USB flash drive (≥1GB).
2. Format it as **FAT32**:
   - **Windows**: File Explorer → Right-click USB → Format → Select FAT32 → Start
   - **Linux**: `sudo mkfs.vfat -F 32 /dev/sdX`
   - **macOS**: `diskutil eraseDisk FAT32 BIOS MBR /dev/diskX`

---

## 📂 Step 2: Extract the Archive

Use a tool like [7-Zip](https://www.7-zip.org/) or `unrar` to extract the archive:

1. Open `BIOS Q670-PLUS unlocked v7.rar`
2. Extract **all contents** directly to the **root of the FAT32-formatted USB stick**

Your USB root must contain the `EFI/` folder, `CW-Q670-PLUS/`, and `auto.nsh`.

---

## ▶️ Step 3: Flash the BIOS

1. Insert the USB stick into the system
2. Power on and boot from the USB stick
4. The system will enter the UEFI shell and automatically execute `auto.nsh`

If successful, the flashing process will begin automatically. Follow any on-screen instructions.

---

## ⚠️ Warning

- Do **not** interrupt power during the flashing process.
- This script may overwrite existing BIOS firmware — use only if you're comfortable flashing.
- It's recommended to reset BIOS settings to default after flashing.

---

## 🔗 Related

- [BIOS Configuration Guide](../configs/bios-settings.md)
