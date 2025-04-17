# 1. Check internet
ping archlinux.org

# 2. Set keyboard (optional)
loadkeys us

# 3. Open disk tool
cfdisk /dev/nvme0n1  # or /dev/sdX for HDDs

# 4. Format partitions
mkfs.fat -F32 /dev/nvme0n1p1        # EFI
mkfs.btrfs /dev/nvme0n1p2           # Root (or ext4 if preferred)
mkswap /dev/nvme0n1p3               # Swap
swapon /dev/nvme0n1p3

# 5. Mount partitions
mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi

# 6. Install base packages
pacstrap /mnt base linux linux-firmware sudo networkmanager git

# 7. Generate fstab
genfstab -U /mnt >> /mnt/etc/fstab

# 8. Change root into system
arch-chroot /mnt

# 9. Set timezone
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc

# 10. Locale
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# 11. Hostname
echo "archsway" > /etc/hostname
cat > /etc/hosts <<EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   archsway.localdomain archsway
EOF

# 12. Set root password
passwd

# 13. Bootloader (systemd-boot)
bootctl install
cat > /boot/loader/entries/arch.conf <<EOF
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=/dev/nvme0n1p2 rw
EOF
echo "default arch" > /boot/loader/loader.conf

# 14. Add user
useradd -m -G wheel -s /bin/bash yourname
passwd yourname
EDITOR=nano visudo
# Uncomment: %wheel ALL=(ALL:ALL) ALL

# 15. Install Sway and required packages
pacman -S sway swaybg swayidle swaylock waybar wl-clipboard alacritty firefox foot grim slurp wofi light brightnessctl pulseaudio pavucontrol

# For NVIDIA users:
# pacman -S egl-wayland

# 16. Configure Sway
mkdir -p ~yourname/.config/sway
cp /etc/sway/config ~yourname/.config/sway/
chown -R yourname:yourname ~yourname/.config

# 17. Enable seat management (for sway)
pacman -S seatd
systemctl enable --now seatd

# 18. Exit & unmount
exit
umount -R /mnt
reboot
