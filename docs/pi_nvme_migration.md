# Raspberry Pi 4: SD Card to NVMe Migration

## Check NVMe Drive

List all block devices to verify NVMe is detected:

```bash
lsblk
```

Expected output shows `sda` (NVMe via USB/HAT) and `mmcblk0` (SD card):

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 119.2G  0 disk
mmcblk0     179:0    0  14.6G  0 disk
├─mmcblk0p1 179:1    0   512M  0 part /boot/firmware
└─mmcblk0p2 179:2    0  14.1G  0 part /
```

Check if NVMe is detected (should show `/dev/sda`):

```bash
ls /dev/sd*
```

## Migration Steps

### 1. Update Bootloader (One-time)

Update system and bootloader:

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install git -y
sudo rpi-eeprom-update -a
sudo reboot
```

### 2. Clone SD to NVMe

Install rpi-clone from GitHub:

```bash
cd ~
git clone https://github.com/billw2/rpi-clone.git
cd rpi-clone
sudo cp rpi-clone rpi-clone-setup /usr/local/sbin
```

Clone to NVMe (`sda` is typically the NVMe drive):

```bash
sudo rpi-clone sda
```

Follow prompts and confirm when asked.

### 3. Fix Boot Configuration

Mount NVMe boot partition:

```bash
sudo mkdir -p /mnt/nvme
sudo mount /dev/sda1 /mnt/nvme
```

Get NVMe root partition PARTUUID (copy the value, e.g., `9dfbe735-02`):

```bash
sudo blkid /dev/sda2
```

Edit cmdline.txt to use NVMe's PARTUUID (change `root=PARTUUID=<old>` to `root=PARTUUID=<sda2-partuuid>`):

```bash
sudo nano /mnt/nvme/cmdline.txt
```

Verify the change (should show `root=PARTUUID=<your-sda2-partuuid>`):

```bash
cat /mnt/nvme/cmdline.txt
```

Unmount:

```bash
sudo umount /mnt/nvme
```

### 4. Enable NVMe Boot Order

Set boot order to prioritize USB/NVMe (change `BOOT_ORDER=0xf14` to `BOOT_ORDER=0xf41` where 4=USB/NVMe, 1=SD card, f=restart):

```bash
sudo -E rpi-eeprom-config --edit
```

Reboot and verify boot order (should show `BOOT_ORDER=0xf41`):

```bash
sudo reboot
sudo rpi-eeprom-config
```

### 5. Boot from NVMe

Power off:

```bash
sudo poweroff
```

Remove SD card physically, then power on. Pi should boot from NVMe.

### 6. Verify Boot from NVMe

Check root filesystem is on `sda2` (NVMe), not `mmcblk0` (SD card):

```bash
lsblk -f
```

Expected output - root (/) mounted on sda2:

```
NAME   FSTYPE LABEL UUID                                 MOUNTPOINTS
sda
├─sda1 vfat         9941-AB36                            /boot/firmware
└─sda2 ext4         8919f609-6b44-49d3-bfc3-e6e5a745c2e9 /
```

SD card can now be safely removed or kept as backup.

## Post-Migration

Once booted from NVMe, use Ansible for all configuration:

- Run `setup_pis.yml` playbook (when created)
- Install k3s cluster
- Configure storage and services
