sudo parted /dev/sda mklabel msdos
sudo parted /dev/sda --script mkpart primary fat32 8MiB 1GiB
sudo parted /dev/sda --script set 1 lba on
sudo parted /dev/sda --script mkpart primary ext4 1GiB 16GiB
sudo mkfs.vfat -F32 -n RPI_BOOT /dev/sda1
sudo mkfs.ext4 -L RPI_FS /dev/sda2

sudo losetup --show -Pf ./Desktop/2025-05-13-raspios-bookworm-armhf-lite.img

sudo mount /dev/loop21p1 /mnt/raspboot
sudo mount /dev/loop21p2 /mnt/rasproot

sudo mount /dev/sda1 /mnt/rpi-boot
sudo mount /dev/sda2 /mnt/rpi-root

sudo rsync -aAXHv --delete /mnt/raspboot/ /mnt/rpi-boot/
sudo rsync -aAXHv --delete /mnt/rasproot/ /mnt/rpi-root/

sudo sync

sudo umount /mnt/rpi-boot /mnt/rpi-root /mnt/raspboot /mnt/rasproot
sudo losetup -d /dev/loop21
