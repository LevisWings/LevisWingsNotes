# Arch Linux Install

### Download



### Install

```bash
loadkeys la-latin1
timedatectl set-ntp true
cfdisk
# Select: dos
# - Select Free Space -> New -> We give 150 GB to the operating system -> Primary
# - Select Free Space -> New -> We give 11GB to the SWAP -> Primary
# - Select Free Space -> New -> We give the remaining GB to the user -> Primary
# Now let's choose the type of each one:
# - The 150GB OS -> Type -> Linux
# - The 11GB of SWAP -> Type -> Linux swap / Solaris
# - The GB left over for the user -> Type -> Linux
# We assign booteable to the OS:
# - The 150GB of the OS -> Booteable
# Finally, we write the partitions and exit.
lsblk # To view the partitions again.
mkfs.ext4 /dev/sda1 # From the operating system.
mkfs.ext4 /dev/sda3 # From the user's home.
mkswap /dev/sda2 # From the swap.
swapon /dev/sda2 # From the swap.
mount /dev/sda1 /mnt # Mount the file system.
mkdir /mnt/home
mount /dev/sda3 /mnt/home

cat /etc/pacman.d/mirrorlist
pacstrap /mnt base linux linux-firmware
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/America/Buenos_Aires /etc/localtime
hwclock --systohc
pacman -S nano
```
