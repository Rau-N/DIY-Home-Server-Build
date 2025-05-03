# 💾 ZFS Configuration with OpenMediaVault

This section documents how the ZFS storage pool is configured inside the OpenMediaVault (OMV) virtual machine with PCIe passthrough and encryption support.

---

## ⚠️ Prerequisite: Enable IOMMU for PCI Passthrough

In order to pass through the SATA controller to the OMV VM, **IOMMU support** must be enabled on the Proxmox host.

---

### 1. Enable IOMMU in Kernel Boot Parameters (systemd-boot)

Edit the kernel command line:

```bash
vi /etc/kernel/cmdline
```

Append or modify the following line:

```
root=ZFS=rpool/ROOT/pve-1 boot=zfs quiet intel_iommu=on iommu=pt
```

Then refresh the bootloader:

```bash
pve-efiboot-tool refresh
```

---

### 2. Load VFIO Kernel Modules

Append the following lines to `/etc/modules`:

```bash
echo "vfio" >> /etc/modules
echo "vfio_iommu_type1" >> /etc/modules
echo "vfio_pci" >> /etc/modules
```

---

### 3. Regenerate initramfs and Reboot

Run:

```bash
update-initramfs -u -k all
systemctl reboot
```

Once rebooted, you can continue with PCI passthrough configuration.

## 1. Pass through the SATA Controller to the OMV VM

To enable SMART and full disk control inside OMV, the SATA controller must be passed through from Proxmox to the VM.

- Go to your OMV VM in Proxmox → **Hardware → Add → PCI Device**

![OMV VM Hardware Overview](../images/omv-vm-hardware.png)

- Select the **SATA controller** from the PCI device list.

![SATA Controller Selected](../images/omv-vm-hardware-sata-controller-selected.png)

![PCI Devices Overview](../images/omv-vm-hardware-pci-devices.png)

- Confirm the device is added under the VM’s hardware list.

![SATA Controller Added](../images/omv-vm-hardware-sata-controller-added.png)



- Start the OMV VM
- Check with `lspci` if the SATA controller gets recognized

```bash
lspci | grep SATA
```

```bash
00:1f.2 SATA controller: Intel Corporation 82801IR/IO/IH (ICH9R/DO/DH) 6 port SATA Controller [AHCI mode] (rev 02)
01:00.0 SATA controller: Intel Corporation Alder Lake-S PCH SATA Controller [AHCI Mode] (rev 11)
```

- Check if the hard drives are detected using:

```bash
lsblk
```

```bash
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   32G  0 disk
├─sda1   8:1    0   31G  0 part /
├─sda2   8:2    0    1K  0 part
└─sda5   8:5    0  975M  0 part [SWAP]
sdb      8:16   0 10,9T  0 disk
├─sdb1   8:17   0 10,9T  0 part
└─sdb9   8:25   0    8M  0 part
sdc      8:32   0 10,9T  0 disk
├─sdc1   8:33   0 10,9T  0 part
└─sdc9   8:41   0    8M  0 part
sdd      8:48   0 10,9T  0 disk
├─sdd1   8:49   0 10,9T  0 part
└─sdd9   8:57   0    8M  0 part
sr0     11:0    1 1024M  0 rom
```

Or via the **OMV Web UI → Storage → Disks**

---

## 2. Install the ZFS Plugin

- Install `omv-extras` from the OMV Web UI or CLI
- Install `openmediavault-zfs` (v7.1.1 or newer)

---

## 3. Create a ZFS Pool

### Unencrypted Pool

Use the **ZFS Plugin** in the OMV Web UI to create a new pool directly.

---

### Encrypted Pool (Manual Creation)

Create the pool from the OMV shell with full options:

```bash
zpool create \
  -o ashift=12 \
  -O encryption=aes-256-gcm \
  -O keyformat=passphrase \
  -O keylocation=prompt \
  -O compression=lz4 \
  -O atime=off \
  -O xattr=sa \
  -O recordsize=128K \
  -O acltype=posixacl \
  zfs_hdd_pool_01 \
  raidz \
  /dev/disk/by-id/ata-WDC_WD120EFBX-68B0EN0_XXXXXXXX \
  /dev/disk/by-id/ata-WDC_WD120EFBX-68B0EN0_XXXXXXXX \
  /dev/disk/by-id/ata-WDC_WD120EFBX-68B0EN0_XXXXXXXX
```

---

## 4. Explanation of ZFS Properties

| Option | Description |
|--------|-------------|
| `ashift=12` | Ensures 4K alignment, which is ideal for modern drives. |
| `compression=lz4` | Enables lightweight compression for better storage efficiency and performance. |
| `atime=off` | Disables atime updates to improve performance by avoiding unnecessary metadata writes. |
| `xattr=sa` | Optimizes extended attribute storage by storing it in system attributes, reducing metadata overhead. |
| `recordsize=128K` | Sets record size for general-purpose workloads and ensures clarity by overriding inherited values. |
| `acltype=posixacl` | Enables support for POSIX ACLs, useful for fine-grained permission control. |


---

## 5. Add Filesystem Entry to OMV Config

Insert a `mntent` entry into `/etc/openmediavault/config.xml`:

```xml
<mntent>
  <uuid>af26493c-d353-11ef-802d-b72d7c92b6e1</uuid>
  <fsname>zfs_hdd_pool_01</fsname>
  <dir>/zfs_hdd_pool_01</dir>
  <type>zfs</type>
  <opts>rw,noatime,nofail,noauto,xattr,posixacl</opts>
  <freq>0</freq>
  <passno>0</passno>
  <hidden>0</hidden>
  <usagewarnthreshold>85</usagewarnthreshold>
  <comment>ZFS HDD Pool</comment>
</mntent>
```

- Use `uuid` command to generate a unique ID:

```bash
uuid
```

- The UUID must be unique across the entire OMV config and will be referenced by SMB shares.

---

## 6. Manual Steps After Reboot

Encrypted ZFS pools need to be unlocked and mounted after reboot:

```bash
# Load the encryption key
zfs load-key zfs_hdd_pool_01

# Mount the pool
zfs mount zfs_hdd_pool_01
```

To automate this, use this existing [script](https://github.com/Rau-N/zfs-key-loader):

```bash
./zfs.sh
```

---
