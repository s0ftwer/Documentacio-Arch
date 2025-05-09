# Documentació tècnica: Guia de instal·lació d'Arch Linux

**Autor:** Álvaro Rodríguez Ruiz
**Curs:** ASIX2B / MP14

---

## Introducció

Arch Linux és un sistema operatiu minimalista pensat per a usuaris que volen tenir control total sobre el seu sistema. A diferència d’altres distribucions com Ubuntu, aquí parteixes de zero, instal·lant només el que realment vols... En resum, estàs construint el teu propi sistema operatiu!

---

## Índex

1. Preparació prèvia
2. Configuració inicial a l'arrencada
3. Particionament del disc amb cfdisk (UEFI)
4. Format i muntatge de les particions
5. Instal·lació del sistema base
6. Configuració del sistema (hora, idioma, hostname)
7. Instal·lació i configuració de GRUB
8. Creació d’usuari i permisos
9. Instal·lació de KDE Plasma
10. VirtualBox Guest Additions
11. Sortida, reinici i comprovacions finals

---

## 1. Preparació prèvia

- Enllaç de ISO oficial de ARCH: https://archlinux.org/download
- Paràmetres de la màquina virtual en VirtualBox:
  - Tipus: Linux, Versió: Arch Linux (64-bit)
  - RAM: 8192 MB
  - Disc dur: 50 GB o més (VDI, dinàmic)

---

## 2. Configuració inicial a l'arrencada

```bash
loadkeys es
ping -c 3 archlinux.org
timedatectl set-ntp true
```

---

## 3. Particionament del disc amb cfdisk (UEFI)

```bash
cfdisk /dev/sda
```

Selecciona **GPT** com a tipus de taula. Crea:
- `/dev/sda1` – 512MB – EFI System
- `/dev/sda2` – resta del disc – Linux filesystem

Escriu i surt.

---

## 4. Format i muntatge

```bash
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2

mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

---

## 5. Instal·lació del sistema base

```bash
pacstrap /mnt base linux linux-firmware networkmanager sudo nano vim
```

```bash
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```

---

## 6. Configuració del sistema

### Hora
```bash
ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
hwclock --systohc
```

### Idioma
```bash
nano /etc/locale.gen
```
Descomenta:
```
es_ES.UTF-8 UTF-8
```

```bash
locale-gen
echo "LANG=es_ES.UTF-8" > /etc/locale.conf
```

### Nom de la màquina
```bash
echo archemetorce > /etc/hostname
nano /etc/hosts
```
Afegir:
```
127.0.0.1   localhost
::1         localhost
127.0.1.1   archemetorce.localdomain archemetorce
```

### Contrasenya root
```bash
passwd
```

---

## 7. Instal·lació de GRUB (UEFI)

```bash
pacman -S grub efibootmgr
mkdir /boot/EFI
mount /dev/sda1 /boot/EFI
grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## 8. Creació d’usuari i permisos sudo

```bash
useradd -m -G wheel alvaro
passwd alvaro
EDITOR=nano visudo
```
Descomenta:
```
%wheel ALL=(ALL:ALL) ALL
```

---

## 9. Instal·lació de KDE Plasma

```bash
pacman -S xorg plasma kde-applications sddm sddm-kcm
systemctl enable NetworkManager
systemctl enable sddm
```

---

## 10. VirtualBox Guest Additions

```bash
pacman -S virtualbox-guest-utils
systemctl enable vboxservice
```

---

## 11. Sortida i reinici

```bash
exit
umount -R /mnt
reboot
```

Assegura’t de treure la ISO de la màquina virtual!

---

## Resultat

Després del reinici, hauries de veure el GRUB, carregar Arch Linux i entrar a KDE Plasma amb l’usuari `alvaro`. Tenim ara un Arch Linux complet i amb el que jo considero el mínim per començar a utilitzar-ho
