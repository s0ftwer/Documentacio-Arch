# Documentació tècnica: Guia d’instal·lació d’Arch Linux

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
12. Comandes bàsiques (Arch vs Ubuntu)
13. Consells per no trencar el sistema

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
loadkeys es                # Canvia el teclat a espanyol
ping -c 3 archlinux.org    # Comprova connexió a internet
timedatectl set-ntp true   # Activa sincronització automàtica de l'hora
```

---

## 3. Particionament del disc amb cfdisk (UEFI)

```bash
cfdisk /dev/sda            # Obre l’eina de particions
```

Selecciona **GPT** com a tipus de taula. Crea:
- `/dev/sda1` – 512MB – EFI System
- `/dev/sda2` – resta del disc – Linux filesystem

Escriu i surt.

---

## 4. Format i muntatge

```bash
mkfs.fat -F32 /dev/sda1    # Dona format FAT32 a la partició EFI
mkfs.ext4 /dev/sda2        # Dona format ext4 al sistema principal

mount /dev/sda2 /mnt       # Munta la partició principal
mkdir /mnt/boot            # Crea carpeta boot
mount /dev/sda1 /mnt/boot  # Munta la partició EFI a boot
```

---

## 5. Instal·lació del sistema base

```bash
pacstrap /mnt base linux linux-firmware networkmanager sudo nano vim
# Instal·la el sistema bàsic i eines essencials
```

```bash
genfstab -U /mnt >> /mnt/etc/fstab  # Genera les taules de particions
arch-chroot /mnt                    # Entra al sistema com si ja estigués instal·lat
```

---

## 6. Configuració del sistema

### Hora

```bash
ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime  # Defineix zona horària
hwclock --systohc                                         # Sincronitza rellotge hardware
```

### Idioma

```bash
nano /etc/locale.gen          # Edita l’arxiu de localitzacions
```
Descomenta:
```
es_ES.UTF-8 UTF-8
```

```bash
locale-gen                                 # Genera les configuracions d’idioma
echo "LANG=es_ES.UTF-8" > /etc/locale.conf
```

### Nom de la màquina

```bash
echo archemetorce > /etc/hostname     # Assigna nom a l’equip
nano /etc/hosts                       # Configura hostnames locals
```

Afegir:
```
127.0.0.1   localhost
::1         localhost
127.0.1.1   archemetorce.localdomain archemetorce
```

### Contrasenya root

```bash
passwd     # Estableix la contrasenya del root
```

---

## 7. Instal·lació de GRUB (UEFI)

```bash
pacman -S grub efibootmgr                      # Instal·la GRUB i eina de boot EFI
mkdir /boot/EFI
mount /dev/sda1 /boot/EFI
grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg          # Genera el fitxer de configuració
```

---

## 8. Creació d’usuari i permisos sudo

```bash
useradd -m -G wheel alvaro      # Crea l’usuari i afegeix-lo al grup sudo
passwd alvaro                   # Assigna contrasenya
EDITOR=nano visudo              # Obre fitxer de sudoers
```

Descomenta:
```
%wheel ALL=(ALL:ALL) ALL
```

---

## 9. Instal·lació de KDE Plasma

```bash
pacman -S xorg plasma kde-applications sddm sddm-kcm  # Instal·la entorn gràfic i gestor d’inici
systemctl enable NetworkManager                       # Activa servei de xarxa
systemctl enable sddm                                 # Activa inici gràfic
```

---

## 10. VirtualBox Guest Additions

```bash
pacman -S virtualbox-guest-utils     # Eines per a millor compatibilitat
systemctl enable vboxservice         # Servei d’integració amb VirtualBox
```

---

## 11. Sortida i reinici

```bash
exit             # Surt del chroot
umount -R /mnt   # Desmunta totes les particions
reboot           # Reinicia la màquina
```

Assegura’t de treure la ISO de la màquina virtual!

---

## 12. Comandes bàsiques (Arch vs Ubuntu)

| Funció                         | Arch Linux (`pacman`)           | Ubuntu/Debian (`apt`)            |
|-------------------------------|----------------------------------|----------------------------------|
| Actualitzar repositoris       | `pacman -Sy`                     | `apt update`                     |
| Actualitzar sistema complet   | `pacman -Syu`                    | `apt upgrade`                    |
| Instal·lar paquet             | `pacman -S <paquet>`             | `apt install <paquet>`           |
| Eliminar paquet               | `pacman -R <paquet>`             | `apt remove <paquet>`            |
| Buscar paquet                 | `pacman -Ss <nom>`               | `apt search <nom>`               |
| Consultar si un paquet està instal·lat | `pacman -Qs <nom>`      | `dpkg -l | grep <nom>`           |
| Neteges de paquets orfes      | `pacman -Rns $(pacman -Qdtq)`    | `apt autoremove`                 |

---

## 13. Consells per no trencar el sistema
- **No fer actualitzacions parcials**: Aquesta és una gran manera d'acabar en l'infern de la dependència i trencar el sistema.
- **No usis `pacman -Sy` sol**: sempre amb `pacman -Syu`, sinó pots provocar inconsistències.
- **Mantén els paquets AUR al mínim**: És fantàstic i en realitat pot evitar que es trenquin les coses si s'entén i s'utilitza correctament, però és una eina perillosa, com qualsevol altra eina poderosa.
- **Evita forçar instal·lacions** amb `--overwrite` o `--force` si no saps el que fas.
- **Llegeix les notícies d’Arch** abans de fer grans actualitzacions (`archlinux.org`).
- **Fes còpies de seguretat**: Els fitxers de configuració que modifiquis sempre han de tindre una copia.
- **Evita eliminar paquets del sistema base** o del grup `base`/`base-devel`.
- **Estigues al dia** Això és menys important, especialment si seguiu les dues primeres regles de prop, però encara és possible tenir problemes si esteu significativament darrere de les actualitzacions
- **Tingues un live USB preparat** per si cal recuperar el sistema.

---

## Resultat

Després del reinici, hauries de veure el GRUB, carregar Arch Linux i entrar a KDE Plasma amb l’usuari `alvaro`. Tens ara un Arch Linux complet i funcional per començar a experimentar!
