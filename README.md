# 🖥️ DIY Home Server Build & Virtualized NAS

This project documents my custom DIY home server and NAS setup — built with energy-efficient and powerful components, using **Proxmox VE** as the hypervisor and virtualized services for storage, media, and applications.

---

## 🧰 Hardware Components

| Component        | Model/Details | Link |
|------------------|---------------|------|
| **Motherboard**  | CWWK Q670-8Bay NAS Mini ITX | [CWWK.net](https://cwwk.net/products/q670-8bay-nas-mini-itx-motherboard-upgraded-version-lga1700-supports-intell12-14-gen-processors-ddr5-dual-4k-displays-5x-usb3-2-8-sata3-0-ports-i226lm-2-5g-with-vpro-q670-2xsff-8643) |
| **CPU**          | Intel Core i5-14500 (14th Gen) | [Amazon](https://www.amazon.de/dp/B0CQ2XT4RT) |
| **RAM**          | Crucial Pro DDR5 96GB (2x48GB) 5600MHz | [Amazon](https://www.amazon.de/dp/B0C79RMMCL) |
| **Power Supply** | HDPLEX 250W Passive GaN AIO PSU | [HDPLEX](https://hdplex.com/hdplex-fanless-250w-gan-aio-atx-psu.html) |
| **NVMe SSDs**    | Samsung 990 PRO 1TB, Samsung 970 EVO Plus 1TB | [990 PRO](https://www.amazon.de/dp/B0B9C3ZVHR) / [970 EVO Plus](https://www.amazon.de/dp/B07MFZY2F2) |
| **HDDs**         | 3x WD Red Plus 12TB (WD120EFBX) | [Amazon](https://www.amazon.de/dp/B08V1L1WYD) |
| **Case**         | Jonsbo N3 | [Jonsbo](https://www.jonsbo.com/en/products/N3.html) |
| **CPU Cooler**   | Noctua NH-L9x65 | [Amazon](https://www.amazon.de/dp/B00VB3Y89E) |
| **Case Fans**    | 2x Noctua NF-A9 FLX (HDD Cooling) | [Amazon](https://www.amazon.de/dp/B00NEMG9B0) |
|                  | 2x Noctua NF-A8 FLX (Motherboard Cooling) | [Amazon](https://www.amazon.de/dp/B00NEMG9K6) |
| **Cables**       | 2x AKYGA AK-CA-11 SATA to Molex Adapter | [Amazon](https://www.amazon.de/dp/B07TWFWQD4) |
|                  | 2x SFF-8643 to 4x SATA Cables | [Amazon](https://www.amazon.de/dp/B00X8ZB63O) |
---

## 🧬 BIOS Configuration

This build uses a **custom BIOS** specifically required for the CWWK Q670 motherboard to unlock key **power-saving features** such as **ASPM (Active State Power Management)**. These features are essential for reducing idle power draw and improving overall energy efficiency.

- The custom BIOS is included in this repository under `/bios/`.
- Flashing instructions and tools are also provided.
- Stock BIOS lacks full ASPM support and limits some PCIe power options.
- Tested and verified for this specific hardware configuration

> ⚠️ **Note:** Flashing the custom BIOS is **optional**. The system works well with the stock BIOS, but flashing is **recommended if maximum power efficiency is desired**.

## ⚙️ Software Configuration

### 🧠 Hypervisor: **Proxmox VE**
- Installed on **ZFS mirror** using:
  - Samsung 990 PRO (1TB) - (M.2 Port 2)
  - Samsung 970 EVO Plus (1TB) - (M.2 Port 3)
- Provides virtualization for multiple services
- Web UI and CLI for full control

---

## 🖥️ Virtual Machines

### 1️⃣ OpenMediaVault (OMV)
- Purpose: **File server / NAS**
- Storage: **SATA controller passthrough** for direct HDD access
- Disk Pool: **ZFS RAIDZ1** (across 3x 12TB HDDs)
- Services:
  - SMB file shares
  - FTP server
  - SMART monitoring

### 2️⃣ Ubuntu Server
- Purpose: **Container host**
- Docker stack includes:
  - **Traefik** (reverse proxy with HTTPS)
  - **Nextcloud** (private cloud)
  - **Jellyfin** (media server)
- **Intel iGPU passthrough** enabled for hardware-accelerated media encoding

---

## 🧪 Technical Highlights

- ✅ **ZFS Mirror** for Proxmox root (resilient boot and data storage)
- ✅ **RAIDZ1** for NAS pool (balance between redundancy and capacity)
- ✅ **PCIe passthrough** for SATA controller (native SMART access + direct disk management)
- ✅ **iGPU passthrough** for Jellyfin transcoding
- ✅ **Fully silent and energy-efficient PSU** and optimized airflow with Noctua fans

---

## 📷 Photos & Diagrams (TBD)
- System internals
- Cable layout
- Proxmox UI + VM configuration screenshots

---

## 📚 Additional Documentation

- 🔐 [ZFS Configuration with OpenMediaVault](docs/zfs-setup.md)