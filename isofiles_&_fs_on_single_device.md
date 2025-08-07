# create new partition table as msdos
```bash
sudo parted /dev/sdX mklabel msdos
```
# create boot & rootfs partitions like rpios image file
```bash
sudo parted /dev/sdX --script mkpart primary fat32 8MiB 512MiB # need to ~100mb for boot files
sudo parted /dev/sdX --script set 1 lba on
sudo parted /dev/sdX --script mkpart primary ext4 512MiB 16GiB # set rpi rootfs partition size
```

# format partitions as fat32 for boot & ex4 for rootfs
```bash
sudo mkfs.vfat -F32 -n RPI_BOOT /dev/sdX1
sudo mkfs.ext4 -L RPI_FS /dev/sdX2
```

# attach rpios image file to virtual loop device
```bash
sudo losetup --show -Pf ./2025-05-13-raspios-bookworm-armhf-lite.img
mkdir -p /mnt/img-boot /mnt/img-root /mnt/rpi-boot /mnt/rpi-root
sudo mount /dev/loopXp1 /mnt/img-boot
sudo mount /dev/loopXp2 /mnt/img-root
```
# mount rpi disk (sdcard)
sudo mount /dev/sdX1 /mnt/rpi-boot
sudo mount /dev/sdX2 /mnt/rpi-root

# safe copy files with rysync from loop device to sdcard 
sudo rsync -aAXHv --delete /mnt/img-boot/ /mnt/rpi-boot/
sudo rsync -aAXHv --delete /mnt/img-root/ /mnt/rpi-root/

# wait copy buffer until done
sudo sync

# lets unmount drives 
sudo umount /mnt/rpi-boot /mnt/rpi-root /mnt/img-boot /mnt/img-root

# set msdos device identifier to rpios image device identifier 
```bash
# get image identifier
sudo dd if=./2025-05-13-raspios-bookworm-armhf-lite.img bs=1 skip=440 count=4 status=none | od -An -tx4
# you will see like this
# bc471543
# lets set this identifier to sdcard with dd
printf '\x43\x15\x47\xbc' | sudo dd of=/dev/sdX bs=1 seek=440 count=4 conv=notrunc
# '\x43\x15\x47\xbc' is reversed "bc471543" hex string
# seek  ; identifier offset
# count ; write sdcard 4 bytes (bc 47 15 43)
# conv  ; set notrunc to prevent disk corruption
# check device id with fdisk
sudo fdisk -l /dev/sdX
# output
# 0xbc471543
```

# remove losetup device
```bash
sudo losetup -d /dev/loopX
```

# partition resizing processes at boot or startup
- /mnt/rpi-root/lib/raspi-config/init_resize.sh   
- /mnt/rpi-root/etc/init.d/resize2fs_once (runs at startup)
- /mnt/rpi-root/etc/rc3.d/S01resize2fs_once (runs at startup, runlevel3)
- /mnt/rpi-root/usr/share/initramfs-tools/scripts/local-premount/firstboot (runs at boot)
- /mnt/rpi-root/usr/share/initramfs-tools/hooks/firstboot (include tools to initramfs for resize partition process while booting)
> Note: if u change a initramfs file u need to update initramfs and copy updated initramfs file to boot partition!!! 
