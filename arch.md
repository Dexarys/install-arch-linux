# Installation de ArchLinux

On récupère l'image ISO officiel [ici](https://www.archlinux.org/download/).
Puis on la grave sur un CD/DVD ou USB.

## Préparation de l'installation

Après le lancement de l'image, on arrive sur "l'installateur" de ArchLinux. 

**1. Changer la langue du clavier**

Par défaut le clavier est en mode US.

```
loadkey fr
```

**2. Vérification du boot mode**

Il existe plusieurs mode de boot "La procédure de démarrage".  
Le BIOS puis l'UEFI qui remplace ce premier petit à petit. D'autres existent mais sont moins courant, dans ce cas il faut se reféré au manuel de la carte mère.
Ces différences entre les deux font qu'il y ai des modifications à apporter lors de l'installation.

```
ls /sys/firmware/efi/efivars
```

Si ce fichier existe, alors le système est en UEFI.


**3. Vérification de la connexion à Internet**

Internet est nécessaire lors de l'installation de ArchLinux. Un ping vérifiera alors si la connexion est présente. Il est conseillé d'utilisé une connexion en ethernet.

```
ping archlinux.org
```

Sinon `wifi-menu` pour lancer la configuration du wifi

**4. Partitionner le disque**

Attention, cette partie peut supprimer toutes les données.

Pour lister les disques disponibles

```
fdisk -l
```

Il est nécessaire de retenir le bon disque pour la suite (souvent représenter par `/dev/sd?`)

Ensuite pour partitionner les disques plusieurs outils sont disponibles : fdisk, parted, cfdisk.
J'utilise pour ma part, `cfdisk`

---

1. Single Boot

* BIOS  

Il faut choisir une partition de type MBR. Attention, le MBR autorise seulement 4 partitions principales, il faut donc créer des partitions étendu pour augmenter le nombre de partitions si nécessaire. 

* UEFI

Il faut choisir une partition de type GPT.

2. Dual Boot

Dans le cadre d'un dual boot avec Windows sur un même disque, il est conseillé de ne pas changé la configuration du disque et préférable d'avoir Windows d'installé en premier.

* BIOS

Créer une partition et la monté avec le tag /boot afin d'y installer le grub plutôt que sur la partition MBR de Windows.

* UEFI

Windows créer une partition [EFI](https://wiki.archlinux.org/index.php/EFI_system_partition) qui peut être utilisé pour le boot de Linux.

---

Ensuite il faut donc créer les partitions nécessaires :

```
/ root directory en ext4
/boot ext4
swap
```

On peut également créer d'autres partitions afin de séparer certains éléments :

```
/home 
/var
```

---

### Informations

Concernant la partition [swap](https://wiki.archlinux.org/index.php/Swap):  
Cette partition permet d'étendre la mémoire vive si celle-ci est insuffisante. Cette partition n'est pas forcément utile si vous possédez déjà suffisament de mémoire vive, cependant elle est obligatoire pour l'hibernation du PC. 

Attention c'est à cette étape qu'il faut mettre en place la [LVM](https://wiki.archlinux.org/index.php/LVM), [chiffrement](https://wiki.archlinux.org/index.php/Disk_encryption) ou [RAID](https://wiki.archlinux.org/index.php/RAID). 

---

**5. Formatter les partitions**

Ensuite, il faut donc formatter les partitions précédemment créer (Attention à bien choisir les bonnes partitions).

Les partitions doivent donc être formatter suivant avec un [système de fichier](https://wiki.archlinux.org/index.php/File_systems). Il en existe plusieurs, tous avec des défauts ou des avantages ([voir en détails](https://en.wikipedia.org/wiki/Comparison_of_file_systems))

On existe ensuite les commandes nécéssaires :

```
mkfs.ext4 /dev/sd??
mkswap /dev/sd??
swapon /dev/sd??
```

**6. Monter les partitions**

Après avoir formatté les partitions, il faut donc les associés au système.

```
mount /dev/sd?? /mnt
mount /dev/sd?? /mnt/boot ou /mnt/boot/efi
```

**7. Installation**

Premièrement, archlinux récupère ses programmes sur des serveurs distants. Par défaut, tous sont activés mais il préférable dans choisir 1 ou 2, de préférences, le plus proche et le plus rapide.

Pour ça il faut modifier le fichier `/etc/pacman.d/mirrorlist`.

Ensuite on peut lancer l'installation des paquets de base : [base](https://www.archlinux.org/groups/x86_64/base/), [base-devel](https://www.archlinux.org/groups/x86_64/base-devel/)

```
pacstrap /mnt base base-devel
```

[TLP](https://wiki.archlinux.org/index.php/TLP) peut également être un paquet utile si l'installation se fait sur un laptop.


**8. Configuration du système**

Tout d'abord, il faut généré le fichier `/etc/fstab` qui liste les partitions du systèmes. 

```
genfstab -U /mnt >> /mnt/etc/fstab
```

On se connecte ensuite au système en tant que root

```
arch-chroot /mnt
```

On est alors connecté au système que l'on vient d'installer en tant que root ce qui permettra de réalisé la première configuration du système avant de le redémarrer. 

* Time zone 

```
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
```

* Localisation

On vérifie que la ligne `fr_FR.UTF-8 UTF-8` et `en_US.UTF-8 UTF-8` n'est pas commenté dans le fichier `/etc/locale.gen`

Puis

```
locale-gen
```
On entre la variable LANG dans le fichier `/etc/locale.conf`

```
LANG=fr_FR.UTF-8
```

On enregistre le type de clavier a utilisé `/etc/vconsole.conf`

```
KEYMAP=fr-latin9
```

* Network

On entre le nom de la machine au sein du fichier `/etc/hostname`

Pas besoin d'entrer dans les détails pour le moment sinon regarder cette [page](https://wiki.archlinux.org/index.php/Network_configuration)

* Mot de passe Root

On doit obligatoirement ajouter un mot de passe au compte root.

```
passwd password
```

On peut terminer la configuration par une dernière commande qui va construire une "image mémoire"

```
mkinitcpio -p linux
```

**9. Configuration du GRUB**

Tout d'abord, que ce soit en dualboot ou singleboot il y a quelques commandes à faire avant de s'attaquer à la configuration du GRUB

On doit installer les paquet suivants

```
pacman -S grub os-prober
pacman -S efibootmgr
```
---

#### Bios
```
grub-install --no-floppy --recheck /dev/sda
```


#### UEFI

```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```
---

```
grub-mkconfig -o /boot/grub/grub.cfg
```
