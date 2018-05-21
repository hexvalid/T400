**For Continue to install**
```
export INSTALL_DISK="/dev/sdb"
export BOOT_PARTITION=${INSTALL_DISK}1
export ROOT_PARTITION=${INSTALL_DISK}2
export HOME_PARTITION=${INSTALL_DISK}3
mount -o discard $ROOT_PARTITION /mnt/gentoo
mount $BOOT_PARTITION /mnt/gentoo/boot
mount -o discard $HOME_PARTITION /mnt/gentoo/home
cp -L /etc/resolv.conf /mnt/gentoo/etc/
mount -t proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --rbind /tmp /mnt/gentoo/tmp
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) $PS1"
```


**Define install disk, root&boot size, mirror**
```
export INSTALL_DISK="/dev/sdb"
export ROOT_SIZE="32897"
export MIRROR_URL="http://ftp.linux.org.tr/gentoo/"
```

**Flashing SSD**
```
hdparm --user-master u --security-set-pass NULL $INSTALL_DISK
hdparm --user-master u --security-erase-enhanced NULL $INSTALL_DISK
partprobe $INSTALL_DISK
```

**Partitioning**
```
parted -a optimal $INSTALL_DISK --script "mklabel gpt"
parted -a optimal $INSTALL_DISK --script "unit mib"
parted -a optimal $INSTALL_DISK --script "mkpart ESP fat32 1 129"
parted -a optimal $INSTALL_DISK --script "set 1 boot on"
parted -a optimal $INSTALL_DISK --script "mkpart primary 129 $ROOT_SIZE"
parted -a optimal $INSTALL_DISK --script "name 2 root"
parted -a optimal $INSTALL_DISK --script "mkpart primary $ROOT_SIZE -1"
parted -a optimal $INSTALL_DISK --script "name 3 home"
export BOOT_PARTITION=${INSTALL_DISK}1
export ROOT_PARTITION=${INSTALL_DISK}2
export HOME_PARTITION=${INSTALL_DISK}3
```

**Creating File Systems**
```
mkfs.fat -F 32 $BOOT_PARTITION
mkfs.ext4 -F -E discard $ROOT_PARTITION
mkfs.ext4 -F -E discard $HOME_PARTITION
```

**Mounting**
```
mount -o discard $ROOT_PARTITION /mnt/gentoo
mkdir -p /mnt/gentoo/boot
mount $BOOT_PARTITION /mnt/gentoo/boot
mkdir -p /mnt/gentoo/boot/EFI
mkdir -p /mnt/gentoo/boot/EFI/gentoo
mkdir -p /mnt/gentoo/home
mount -o discard $HOME_PARTITION /mnt/gentoo/home
```

**Downloading stage**
```
export LASTEST_STAGE=$(curl -s ${MIRROR_URL}releases/amd64/autobuilds/latest-stage3-amd64.txt | tail -n 1 | cut -d " " -f 1)
wget ${MIRROR_URL}releases/amd64/autobuilds/$LASTEST_STAGE -O /tmp/latest-stage3-amd64.tar.bz2
```

**Unpacking stage**
```
tar xpf /tmp/latest-stage3-amd64.tar.bz2 --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo/ && sync
```

**Configs**
#...

**Chroot**
```
cp -L /etc/resolv.conf /mnt/gentoo/etc/
mount -t proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --rbind /tmp /mnt/gentoo/tmp
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) $PS1"
```

**Init portage**
```
emerge-webrsync
emerge --sync
eselect news list 
eselect news read
```

**World**
```
emerge --ask --update --deep --newuse @world
```
