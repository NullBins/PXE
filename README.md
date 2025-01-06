<div align ="center">
    <h1> [ PXE 🖥 BootLoader 🗂 FileSystem Server Configuration ] </h1>
</div>

>![Image](https://github.com/NullBins/PXE/blob/main/IMAGES/PXE.png)

## [ *Server - Step. 1* ]

📁Server Side - Add to Disk (30GB, sdb)

```bash
apt install lvm isc-dhcp-server nfs-kernel-server tftpd-hpa syslinux syslinux-efi
sed -i 's/v4=""/v4="ens33"/g' /etc/default/isc-dhcp-server
```
```vim
vim /etc/dhcp/dhcpd.conf
```
>```vim
>subnet 192.168.1.0 netmask 255.255.255.0 {
>  range 192.168.1.150 192.168.1.200;
>  option routers 192.168.1.254;
>  option domain-name "vdi.local";
>  option domain-name-servers 192.168.1.1;
>  next-server 192.168.1.1;
>  filename "syslinux.efi";
>  option root-path "192.168.1.100:/pxe/client/pc1";
>}
>```
```bash
systemctl restart isc-dhcp-server
```
```vim
pvcreate /dev/sdb
vgcreate data /dev/sdb
lvcreate -l 100%FREE -n sysvol data /dev/sdb
mkfs.ext4 /dev/data/sysvol
mkdir -p /pxe/client/pc1
```
```vim
vim /etc/fstab
```
>```vim
>/dev/data/sysvol  /pxe/  ext4  defaults,_netdev  0  0
>```
```bash
mount -a
```
```vim
vim /etc/exports
```
>```vim
>/pxe/client/    192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
>```
```bash
systemctl restart nfs-kernel-server
```
```vim
vim /etc/default/tftpd-hpa
```
>```vim
>TFTP_DIRECTORY="/srv/tftp/"
>TFTP_ADDRESS="0.0.0.0:69"
>TFTP_OPTIONS="--secure --create"
>```
```bash
systemctl restart tftpd-hpa
```
```vim
cp -ax /usr/lib/SYSLINUX.EFI/efi64/syslinux.efi /srv/tftp/
cp -r /usr/lib/syslinux/modules/efi64/* /srv/tftp/
mkdir -p /srv/tftp/pxelinux.cfg/
```
```vim
vim /srv/tftp/pxelinux.cfg/default
```
>```vim
>default menu.c32
>prompt 0
>menu title PXE BOOTLOADER SERVICE MENU
>
>label 1
>  menu label [VDI-LOCAL] Client-PC
>  kernel vmlinuz.pxe
>  append rw initrd=initrd.pxe root=/dev/nfs nfsroot=192.168.1.1:/pxe/client/pc1/
>```
```vim
vim /etc/crontab
```
>```vim
>@reboot  root  sleep 20;  systemctl restart nfs-kernel-server
>```
　
📌 클라이언트 PC에서 Display, CPU, RAM, LAN-Card만 남기고 제거 후 서버에서 제거 된 클라이언트 PC디스크(sdc) 추가
　
## [ *Server - Step. 2* ]
```bash
mount /dev/sdc2 /mnt/          👉 # 리눅스 클라이언트 파일시스템 부분임 (fdisk -l 명령어로 확인) #
```
```vim
cp -ar /mnt/* /pxe/client/pc1/
umount /mnt/
cd /pxe/client/pc1/
```
```vim
vim ./etc/fstab
```
>```vim
>/dev/nfs    /      nfs    tcp,nolock    0    0
>proc        /proc  proc   defaults      0    0
>```
```vim
vim ./etc/initramfs-tools/initramfs.conf
```
>```vim
>(VIM Number 73) BOOT=nfs
>```
```vim
mount --rbind /dev/ ./dev/
mount --rbind /proc/ ./proc/
mount --rbind /sys/ ./sys/
chroot /pxe/client/pc1/ /bin/bash --login
```
```bash
chroot /pxe/client/pc1/ /bin/bash --login    👉 # 클라이언트 PC 디스크(/pxe/client/pc1/) 파일 시스템을 bash shell로 로그인
```
```vim
mkinitramfs -d /etc/initramfs-tools -o /boot/initrd.pxe
update-initramfs -u
cp -vax /boot/initrd.img-4.19.0-14-amd64 /boot/initrd.pxe
cp -vax /boot/vmlinuz-4.19.0-14-amd64 /boot/vmlinuz.pxe
exit
cp -ax /pxe/client/pc1/boot/*.pxe /srv/tftp/
```
## [ *Client - Removed Disks* ]
🔔 Booting (Power-On)
>![Image](https://github.com/NullBins/PXE/blob/main/IMAGES/LOGIN.png)
