# Tutorial Instalasi Arch Linux

Kunjungi tutorial resmi [di sini](https://wiki.archlinux.org/title/Installation_guide) 

- Download the official image file from official site.
- Membuat bootable
- Boot with thah

Jika anda melihat ini :
```
[ root@archiso `]$ 
```
Maka anda siap mengikuti tutorial ini.

## Bagian 1: Pre-Installation 

### Tahap 1: Kostumisasi virtual console
* Gunakan `loadkeys` untuk mengatur keyboard, contoh untuk *US*
```
# loadkeys us
```
* Mengubah ukuran font :
```
# setfont ter-132b
```
### Tahap 2: Menghubungkan ke internet
Network device configuration
```
# ip link
```
Pilihan untuk menghubungkan ke internet :
- Ethernet
- Wi-Fi
- Mobile broadband modem
Verifikasi internet `ping 8.8.8.8`, jika output nya ada yang seperti ini : `Name or service not known` maka anda belum terhubung ke internet.
Cek waktu sistem `# timedatectl`
### Tahap 3: Mempartisi disk
Identifikasi [block device](https://wiki.archlinux.org/title/Device_file#Block_devices) menggunakan `lsblk`
Gunakan `cfdisk` untuk mengatur partisi.
Buatlah partisi sesuai kebutuhan. Misal
1. Root 
2. Home 
3. Boot 
4. Swap 
### Tahap 4: Format dan Mount Partisi
Partisi *root* :
```
# mkfs.ext4 /dev/root_partition
# mount /dev/root_partition /mnt
```
Partisi *home* : 
```
# mkfs.ext4 /dev/home_partition
# mkdir -p /mnt/home
# mount /dev/home_partition /mnt/home
```
Untuk partisi *swap*
```
# mkswap /dev/swap_partition
# swapon /de/swap_partition
```
#### Boot 
Jalankan `ls /sys/firmware/efi/efivars`, jika terdapar direktori maka boot mode nya adalah *UEFI*
Bagi pengguna dengan sistem *EFI* format dengan :
```
# mkdir /mnt/boot/efi
# mkfs.fat -F 32 /dev/efi_partition
# mount /dev/efi_partition /mnt/boot/efi
```
Bagi pengguna dengan sistem *MBR* format dengan 
```
# mkfs.ext4 /dev/boot_partition
# mkdir -p /mnt/Boot
# mount /dev/boot_partition /mnt/boot
```
verifikasi boot flag
```
# parted /dev/sda
# set 1 boot on 
```

## Bagian kedua: Installation

* mengatur mirror, edit `/etc/pacman.d/mirrorlist`

### Tahap 1: Install package 
Package wajib di install : `base`, `linux`, `linux-firmware`, opsional: `linux-lts`
Package lain yang juga penting
- hardare bug and security fixes : `amd-ucode`, `intel-ucode`
- Boot loader : `grub` (untuk efi sistem), `syslinux`
- Alat untuk membuild packages linux : `base-devel`
- Network manager : `networkmanager`
- text editor untuk console : `vim` or `nano`
Gunakan `pacstrap` untuk mengunduh packages, contoh :
```
pacstrap -K /mnt base
```
## Bagian ketiga: Mengkonfigurasi sistem
* membuat [fstab](https://wiki.archlinux.org/title/Fstab) file : 
```
# genfstab -U /mnt >> /mnt/etc/fstab
```
cek lagi dengan : `# cat /mnt/etc/fstab`
opsional : edit UUID sesuai partisi
### Konfigurasi root
Ganti ke root :
```
# arch-chroot /mnt
```
Hal yang akan dilakukan
1. Mengatur time zone 
2. Mengatur pengaturan bahasa (locale)
3. Menambahkan pengguna / user
4. Mengatur bootloader
```
# arch-chroot /mnt
```
* Mengatur waktu
```
# ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
# hwclock --systohc
```
* locale, edit `/etc/locale.gen` dan hapus komentar untuk `en_US.UTF-8 UTF-8`, kemudian jalankan :
```
# locale-gen
```
Buat `/etc/locale.conf`, dan tulis :
```
LANG=en_US.UTF-8
```
opsional : Mengatur keyboard layout, edit `/etc/vconsole.conf` :
```
KEYMAP=us
```
cek lagi dengan `# mkinitcpio -p linux-lts`

### Mengatur User
Membuat nama host, edit `/etc/hostname` kemudian tulis nama host nya, setelah itu atur password dengan `passwd`
Menambahkan user, contoh :
```
useradd -m -G wheel -s /bin/bash name
```
edit `VISUDO`

#### Mengatur bootloader
Untuk *UEFI* sistem, gunakan `grub` 
```
# grub-install 
# grub-mkconfig -o /boot/grub/grub.cfg
```
Untuk *LEGACY*, gunakan `syslinux`
```
# pacman -S syslinux
# syslinux-install_update -i -a -m
```

## Bagian akhir: Reboot
Sebelum rebot, aktifkan service penting.

### Unmount
```
umount -R /mnt
```
or
```
umount all
```

reboot dengan `reboot`

Reference
[LEGACY SETUP](https://gist.github.com/xbns/cb8d0f9734a99c19c2503d8439f79e71#file-arch-linux-installation-on-mbr-system-md)
[UEFI SETUP](https://gist.github.com/xbns/3516ee4582f74fc3c41fee3541369fd5#file-arch-linux-installation-on-uefi-gpt-system-md)