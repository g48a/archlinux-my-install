## Full install with UEFI && Encryption, Mostly I do this:


 
### Step 0 - Disk Partitioning
<br/>

**fdisk /dev/vda** <br/>

**/dev/vda1** &nbsp;&nbsp;&nbsp; # EFI System type_id: 1 ***Last Sector +512M***<br/>
**/dev/vda2** &nbsp;&nbsp;&nbsp; # BOOT type_id: SKIP_THIS_ONE ***Last Sector+512M***<br/>
**/dev/vda3** &nbsp;&nbsp;&nbsp; # LVM type_id: 43 ***Last Sector 100%FREE***<br/>
<br/>

### Step 1 - Making File System

`After this we need to make our filesystem`<br/>
**mkfs.fat -F32 /dev/vda1**&nbsp;&nbsp;&nbsp;&nbsp;`# for EFI`<br/>
**mkfs.ext4 /dev/vda2** &nbsp;&nbsp;&nbsp;&nbsp;`# for the future BOOT`<br/>
#### Step 1.1 - Enctyption part
`# And we also need to encrypt our /dev/vda3`<br/>
cryptsetup luksFormat /dev/vda3 <br/>
`<Make your self a good pass>` <br/>

`Now we need to unlock it`<br/>
cryptsetup open --type luks /dev/vda3 lvm<br/>
#### Step 1.2 - LVM partition
pvcreate --dataalignment 1m /dev/mapper/lvm<br/>
vgcreate volgroup0 /dev/mapper/lvm<br/>
lvcreate -L 30GB volgroup0 -n lv_root<br/>
lvcreate -l 100%FREE volgroup0 -n lv_home<br/>
`Now we need to activate it` <br/>
modprobe dm_mod<br/>
vgscan<br/>
vgchange -ay<br/>
`Formatting Logical Volume` <br/>
mkfs.ext4 /dev/volgroup0/lv_root<br/>
mount /dev/volgroup0/lv_root &nbsp;&nbsp;/mnt<br/>

`boot` <br/>
mkdir /mnt/boot<br/>
mount /dev/vda2 &nbsp;&nbsp;/mnt/boot<br/>

`home` <br/>
mkfs.ext4 /dev/volgroup0/lv_home<br/>
mkdir /mnt/home<br/>
mount /dev/volgroup0/lv_home &nbsp;&nbsp;/mnt/home<br/>

`moving genfstab` <br/>
mkdir /mnt/etc<br/>
genfstab -U -p /mnt >> /mnt/etc/fstab<br/>
cat /mnt/etc/fstab<br/>


### Step 2 - Installation section
pacstrap -i /mnt base<br/>
`if you got invalid or corrupted package (PGP signature, then type this and repead previous command)`<br/>
pacman -Syu archlinux-keyring<br/>
`if it's too large then type pacman -S instead of pacman -Syu`<br/>
arch-chroot /mnt<br/>
`Two kernels better than one, one is broken, second up and running`<br/>
pacman -S linux linux-headers linux-lts linux-lts-headers<br/>
`You need text editor`<br/>
pacman -S nano<br/>
pacman -S base-devel openssh<br/>
systemctl enable sshd<br/>
pacman -S networkmanager wpa_supplicant wireless_tools netctl<br/>
pacman -S dialog<br/>
systemctl enable NetworkManager<br/>
pacman -S lvm2<br/>


`Change conf file`<br/>
nano /etc/mkinitcpio.conf<br/>
`Find this line:`<br/>
Find HOOKS=(base udev autodetect modconf block filesystems keyboard fsck)<br/>
`And change it to`<br/>
Find HOOKS=(base udev autodetect modconf block encrypt lvm2 filesystems keyboard fsck)<br/>

mkinitcpio -p linux<br/>
mkinitcpio -p linux-lts<br/>
`FIND and UNCOMMENT en_US.UTF-8 UTF-8`<br/>
nano /etc/locale.gen<br/>
locale-gen<br/>
passwd<br/>
`<create password for your root>`<br/>
useradd -m -g users -G wheel (your_username_goes_here)<br/>
passwd (your_user_name)<br/>
`<create password for your user>`<br/>
pacman -S sudo<br/>
which sudo<br/>
EDITOR=nano visudo<br/>
`FIND AND UNCOMMENT`<br/>
%wheel ALL=(ALL) ALL<br/>


### Step 3 - GRUB Installation
pacman -S grub efibootmgr dosfstools os-prober mtools<br/>
mkdir /boot/EFI<br/>
mount /dev/vda1 /boot/EFI<br/>
exit<br/>
systemctl daemon-reload<br/>
arch-chroot /mnt<br/>
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck<br/>
`Check if locale folder exists`<br/>
ls -l /boot/grub<br/>
`If it is not, then create it by mkdir /boot/grub/locale`<br/>
cp /usr/share/locale/en\\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo<br/>
`We need to inform grub to unlock the disk at boot`<br/>
nano /etc/default/grub<br/>
`Find and uncomment GRUB_ENABLE_CRYPTODISK=y and then find `<br/>
`Now find this one GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"`<br/>
`and change it to  GRUB_CMDLINE_LINUX_DEFAULT="cryptdevice=/dev/vda3:volgroup0:allow-discards loglevel=3 quiet nomodeset"`<br/>
grub-mkconfig -o /boot/grub/grub.cfg<br/>

`Now you can reboot your PC, Congratulations!`:partying_face::partying_face:<br/>
exit<br/>
umount -a<br/>

reboot:orangutan:<br/>


### Step 4 - Post-install Tweaks
`If you want to create swap file, you are allowed to, thanks to LVM partiotion type`<br/>
su<br/>
cd /root<br/>
dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress<br/>
chmod 600 /swapfile<br/>
mkswap /swapfile<br/>
cp /etc/fstab /etc/fstab.bak<br/>
echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab<br/>
cat /etc/fstab<br/>
free -m<br/>
mount -a<br/>
free -m<br/>
swapon -a<br/>

timedatectl list-timezones<br/>
timedatectl set-timezone US/Eastern<br/>
systemctl enable systemd-timesyncd<br/>

hostnamectl set-hostname computer<br/>
cat /etc/hostname<br/>
nano /etc/hosts<br/>
127.0.0.1 localhost<br/>
127.0.1.1 computer<br/>

`Now it is time to install MicroCode for your CPU! or amd or intel`<br/>
pacman -S amd-ucode || pacman -S intel-ucode<br/>
pacman -S xorg-server `But you can also install Wayland instead of Xorg`<br/>
`Video-Driver GPU from Intel or AMD`<br/>
pacman -S mesa<br/>
`If GPU from nvidia, you need to install it for each linux kernel, we have two of them so`<br/>
pacman -S nvidia nvidia-lts
`If you are running this through VM, then install this and next one`
pacman -S virtualbox-guest-utils xf86-video-vmware
systemctl enable vboxservice


### Step 5 - Installing Desktop Environmental (bspwn)
pacman -S xorg-xinit bspwn sxhkd dmenu nitrogen picom xfce4-terminal firefox arandr

mkdir  .config/bspwm
mkdir  .config/sxhkd
cp /usr/share/doc/bspwm/examples/bspwmrc .config/bspwm
cp /usr/share/doc/bspwm/examples/sxhkdrc .config/sxhkd

`Find line #terminal emulator and write your terminal`
nano .config/sxhkd/sxhkdrc
```
# terminal emulator
super + Return
&nbsp;&nbsp;&nbsp;xfce4-terminal
```
