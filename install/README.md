# Tutorial Instalasi Arch Linux

Kunjungi tutorial resmi [di sini](https://wiki.archlinux.org/title/Installation_guide) 

Sebelum mengikuti tutorial ini, pastikan anda telah melakukan hal berikut
- Download official [image](https://archlinux.org/download/) file.
- Membuat bootable 
- Boot ke bootable

Jika anda melihat ini di komputer :

```
[ root@archiso `]$ 
```

Maka anda siap mengikuti tutorial ini.

## Monfigurasi virtual console
- Gunakan `loadkeys` untuk mengatur [keymap](https://wiki.archlinux.org/title/Linux_console/Keyboard_configuration), contoh untuk *United State English*. Untuk melihat keymap yang tersedia, gunakan `$ localectl list-keymaps`

```
$ loadkeys us
```

## Menghubungkan ke internet
Proses instalasi Arch linux membutuhkan koneksi internet. Beberapa pilihan untuk menghubungkan ke internet :
- Ethernet : plug in cable
- Wi-Fi : use `iwctl`
- Mobile broadband modem : use `nmcli`

Untuk memverifikasi internet, gunakan `$ ping 8.8.8.8`, jika output nya ada yang seperti ini : `Name or service not known` maka komputer belum terhubung ke internet.

## Partitioning, Formatting, Mouting
Identifikasi [file perangkat](https://wiki.archlinux.org/title/Device_file#Block_devices) menggunakan `lsblk`.  Terdapat dua tipe dalam [partition table](https://wiki.archlinux.org/title/Partitioning), yaitu `MBR / MASTER BOOT RECORD` dan `GPT / GUID PARTITION TABLE`. Pastikan anda mengetahui tipe yang dimiliki komputer.    
Gunakan `cfdisk` untuk mengatur partisi.
Buatlah partisi sesuai kebutuhan. Misalnya : 
1. Partisi boot 
2. Partisi root 
3. Partisi home 
4. Partisi swap
5. Partisi lainnya: data, var

- Partisi boot
Bagi pengguna dengan sistem *GPT* :
```
$ mkfs.fat -F 32 /dev/efi_partition
$ mount --mkdir /dev/efi_partition /mnt/boot/efi
```
Bagi pengguna dengan sistem *MBR* format dengan 
```
$ mkfs.ext4 /dev/boot_partition
$ mount --mkdir /dev/boot_partition /mnt/boot
```
Setelah itu atur boot flag pada boot partisi :
```
$ parted /dev/sda
parted$ set 1 boot on 
```

- Partisi *root* :
Format dengan :
```
$ mkfs.ext4 /dev/root_partition
```
Kemudian mount:
```
$ mount /dev/*root_partition* /mnt
```
- Partisi *home* : 
Format dengan : `$ mkfs.ext4 /dev/*home_partition*`
Kemudian mount `$ mount --mkdir /dev/home_partition /mnt/home`

- Jika menggunakan partisi *swap*, format dengan `$ mkswap /dev/swap_partition`, kemudian jalankan `$ swapon /de/swap_partition`

## Installation

Pada bagian ini, kita akan mendownload dan menginstall beberapa software yang dibutuhkan. Gunakan `pacstrap -K /mnt` di ikuti nama package yang akan di install.

- Wajib di install : `base`, `linux`, `linux-firmware`, opsional: `linux-lts`
Package lain yang juga penting
- Hardare bug and security fixes : `amd-ucode` atau `intel-ucode`
- [Boot loader](https://wiki.archlinux.org/title/Arch_boot_process) : `grub` (untuk *UEFI* sistem), `syslinux` (untuk *BIOS* sistem)
- Alat untuk membuild packages linux : `base-devel`
- Network manager : `networkmanager`
- text editor untuk console : `vim` atau `nano`


Setelah itu, kita perlu membuat [fstab](https://wiki.archlinux.org/title/Fstab) : 
```
& genfstab -U /mnt >> /mnt/etc/fstab
```
Cek hasilnya dengan menjalankan `$ cat /mnt/etc/fstab`. Pastikan hasilnya sesuai dengan partisi yang telah kita buat.
opsional : edit *UUID* pada fstab file sesuai partisi

## Konfigurasi root
Ganti ke root sistem baru : `$ arch-chroot /mnt`
Hal yang akan dilakukan
1. Mengatur time zone 
2. Mengatur pengaturan bahasa (locale)
3. Menambahkan pengguna / user
4. Mengatur bootloader

* Mengatur [zona waktu](https://wiki.archlinux.org/title/System_time#Time_zone)
```
$ ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
$ hwclock --systohc
```
- Mengatur locale, edit `/etc/locale.gen` dan hapus komentar untuk `en_US.UTF-8 UTF-8`, kemudian jalankan : `$ locale-gen`

- Buat file `/etc/locale.conf`, dan tulis : `LANG=en_US.UTF-8`
opsional : Mengatur keyboard layout, edit `/etc/vconsole.conf` dan tulis
```
KEYMAP=us
```

Setelah itu jalankan `$ mkinitcpio -p linux-lts`

### Menambahkan [pengguna](https://wiki.archlinux.org/title/Users_and_groups#User_management)
Membuat nama host, edit `/etc/hostname` kemudian tulis nama host nya, setelah itu atur password dengan `passwd`
Menambahkan user, contoh :
```
useradd -m -G wheel -s /bin/bash dapz
```
edit `VISUDO` dengan `$ EDITOR=vim visudo`. Kemudian hapus pada komentar pada bagian `wheel group`

#### Mengatur bootloader
Untuk *UEFI* sistem, gunakan `grub` 
```
# grub-install 
# grub-mkconfig -o /boot/grub/grub.cfg
```
Untuk *BIOS*, gunakan `syslinux`
```
# pacman -S syslinux
```
copy `/usr/share/syslinux/syslinux.cfg` to `/boot/syslinux/syslinux.cfg`
```
# syslinux-install_update -i -a -m
```
Jika sudah aman, keluar dari root dengan `$ exit`
## Bagian akhir: Reboot
Sebelum rebot, aktifkan service penting.

Umount dengan menggunakan : `$ umount -R /mnt` atau `$ umount all`

Lalu reboot komputer dengan `reboot` atau jika ingin mematikan komputer gunakan `poweroff`

[Tutorial lainnya]()

Reference
[LEGACY SETUP](https://gist.github.com/xbns/cb8d0f9734a99c19c2503d8439f79e71#file-arch-linux-installation-on-mbr-system-md)
[UEFI SETUP](https://gist.github.com/xbns/3516ee4582f74fc3c41fee3541369fd5#file-arch-linux-installation-on-uefi-gpt-system-md)
