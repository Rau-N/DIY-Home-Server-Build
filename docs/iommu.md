# 🧩 Enabling IOMMU on Proxmox (Required for PCI Passthrough)

IOMMU must be enabled on the Proxmox host in order to use **PCI passthrough** — whether for SATA controllers, GPUs, or other hardware.

---

## 1. Edit Kernel Command Line (systemd-boot)

Open the command line file:

```bash
vi /etc/kernel/cmdline
```

Modify the line to include:

```
root=ZFS=rpool/ROOT/pve-1 boot=zfs quiet intel_iommu=on iommu=pt
```

Apply the changes:

```bash
pve-efiboot-tool refresh
```

---

## 2. Load Required VFIO Modules

Append the following to `/etc/modules`:

```bash
vfio
vfio_iommu_type1
vfio_pci
```

---

## 3. (Optional) Blacklist Host Drivers

Only do this if the device (e.g., SATA controller) must be detached from the host:

```bash
echo "blacklist ahci" >> /etc/modprobe.d/blacklist.conf
```

This prevents the host from binding the device before VFIO claims it.

---

## 4. Regenerate Initramfs and Reboot

Run the following:

```bash
update-initramfs -u -k all
systemctl reboot
```

---

## 🔗 Used in:

- [ZFS Configuration with OpenMediaVault](./zfs-setup.md)
- [Intel iGPU Passthrough to Ubuntu VM](./igpu-passthrough.md)
