# Commande Utile

[![Visit Repository](https://img.shields.io/badge/Visit-Commande--Utile-orange?style=for-the-badge&logo=github)](https://github.com/Tekaya1/linux)

# Exercices de Gestion LVM

## Exercice 1 : Opérations de base sur LVM

### 1.1 Création et gestion des volumes logiques
```bash
# Créer des partitions
fdisk /dev/sdb
# Créer /dev/sdb1 (1G) et /dev/sdb2 (2G)

# Initialiser les volumes physiques
pvcreate /dev/sdb1 /dev/sdb2

# Créer un groupe de volumes
vgcreate vg1 /dev/sdb1 /dev/sdb2 -s 16M

# Créer et configurer LV1 (25 LE, ext4, montage temporaire)
lvcreate -n lv1 -l 25 vg1
mkfs.ext4 /dev/vg1/lv1
mkdir /lv1
mount /dev/vg1/lv1 /lv1

# Créer et configurer LV2 (500M, xfs, montage permanent)
lvcreate -n lv2 -L 500M vg1
mkfs.xfs /dev/vg1/lv2
mkdir /lv2
echo "/dev/vg1/lv2 /lv2 xfs defaults 0 0" >> /etc/fstab
mount -a

# Créer et configurer LV3 (300M, swap)
lvcreate -n lv3 -L 300M vg1
mkswap /dev/vg1/lv3
echo "/dev/vg1/lv3 swap swap defaults 0 0" >> /etc/fstab
swapon -a
```

### 1.2 Opérations de redimensionnement
```bash
# Étendre LV2 de 15 LE en considérant le système de fichiers
lvextend -l +15 /dev/vg1/lv2
xfs_growfs /dev/vg1/lv2

# Réduire LV1 de 100M en considérant le système de fichiers
umount /lv1
resize2fs /dev/vg1/lv1 400M
lvreduce -L 400M /dev/vg1/lv1
mount /dev/vg1/lv1 /lv1

# Étendre LV2 de 2G (ajouter de nouveaux volumes physiques si nécessaire)
lvextend -L +2G /dev/vg1/lv2
xfs_growfs /dev/vg1/lv2
```

### 1.3 Suppression et nettoyage
```bash
umount -a
swapoff -a
lvremove /dev/vg1/lv{1,2,3}
vgremove vg1
pvremove /dev/sdb{1,2}
```

---

## Exercice 2 : Volumes VFAT et XFS

### 2.1 Création et gestion des volumes logiques
```bash
# Créer un groupe de volumes avec 100 PE (32M chacun)
vgcreate vg-lab /dev/sdb1 -s 32M

# Créer et configurer LV1 (70 LE, vfat, montage permanent)
lvcreate -n lv1 -l 70 vg-lab
mkfs.vfat /dev/vg-lab/lv1
mkdir /mnt/lv1
echo "/dev/vg-lab/lv1 /mnt/lv1 vfat defaults 0 0" >> /etc/fstab
mount -a

# Créer et configurer LV2 (30 LE, xfs, montage temporaire)
lvcreate -n lv2 -l 30 vg-lab
mkfs.xfs /dev/vg-lab/lv2
mkdir /mnt/lv2
mount /dev/vg-lab/lv2 /mnt/lv2
```

### 2.2 Opérations de redimensionnement
```bash
# Étendre LV2 de 10 LE
lvextend -l +10 /dev/vg-lab/lv2
xfs_growfs /dev/vg-lab/lv2
```

### 2.3 Suppression et nettoyage
```bash
umount -a
lvremove /dev/vg-lab/lv{1,2}
vgremove vg-lab
```

---

## Exercice 3 : Groupe de volumes "data"

### 3.1 Création et gestion des volumes logiques
```bash
# Créer un groupe de volumes avec 35 PE (32M chacun)
vgcreate data /dev/sdb1 -s 32M

# Créer et configurer LV1 (317M, xfs, montage permanent)
lvcreate -n lv1 -L 317M data
mkfs.xfs /dev/data/lv1
mkdir /lv1
echo "/dev/data/lv1 /lv1 xfs defaults 0 0" >> /etc/fstab
mount -a

# Créer et configurer LV2 (10 LE, ext4, montage temporaire)
lvcreate -n lv2 -l 10 data
mkfs.ext4 /dev/data/lv2
mkdir /lv2
mount /dev/data/lv2 /lv2
```

### 3.2 Opérations de redimensionnement
```bash
# Étendre LV1 de 5 LE sans considérer le système de fichiers
lvextend -l +5 /dev/data/lv1

# Aligner le système de fichiers à la taille de LV1
xfs_growfs /dev/data/lv1

# Étendre LV2 de 13M sans considérer le système de fichiers
lvextend -L +13M /dev/data/lv2

# Aligner le système de fichiers à la taille de LV2
resize2fs /dev/data/lv2

# Réduire LV2 de 3 LE
lvreduce -l -3 /dev/data/lv2 -r
```

### 3.3 Suppression et nettoyage
```bash
umount -a
lvremove /dev/data/lv{1,2}
vgremove data
```

---

## Exercice 4 : Configuration de SWAP et EXT4

### 4.1 Création et gestion des partitions
```bash
# Créer une partition SWAP (512M)
fdisk /dev/sdb
mkswap /dev/sdb1
echo "/dev/sdb1 swap swap defaults 0 0" >> /etc/fstab
swapon -a

# Créer une partition de 1G formatée en EXT4 (montage permanent)
fdisk /dev/sdb
mkfs.ext4 /dev/sdb2
mkdir /part
echo "/dev/sdb2 /part ext4 defaults 0 0" >> /etc/fstab
mount -a
```

### 4.2 Création et gestion des volumes logiques
```bash
# Créer un groupe de volumes avec 60 PE
pvcreate /dev/sdb3
vgcreate group /dev/sdb3 -s 17M

# Créer et configurer LV (200M, ext4, montage permanent)
lvcreate -n lv -L 200M group
mkfs.ext4 /dev/group/lv
mkdir /volume
echo "/dev/group/lv /volume ext4 defaults 0 0" >> /etc/fstab
mount -a
```

### 4.3 Opérations de redimensionnement
```bash
# Étendre LV de 300M
lvextend -L +300M /dev/group/lv
resize2fs /dev/group/lv
```
