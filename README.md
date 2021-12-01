# grub-2.04-nokeymod
Credit/license/source:  Originally obtained from grub-2.04.tar.gz at https://git.savannah.gnu.org/cgit/grub.git 
Forked from: https://github.com/coherixmatts/grub-2.04

This fork changes the timeout behaviour of GRUB to allow selecting an OS on a Surface RT device using the volume buttons only. The timeout will continue until 0 and then boot the selected OS.

## Requirements
- Download the grub.efi file from the release section or compile it yourself.
- On the EFI partition (or USB drive formatted as FAT32) create the folders EFI\boot
- Place the grub.efi into the EFI\boot folder and rename it to bootarm.efi
- Download and place the file unicode.pf2 in the root of the partition.
- Create a grub.cfg config file and place it at the root of the partition. An example can be found below.
- (Optional:) Place all kernel and device tree files you need to boot linux in the root of the partition

## grub.cfg
Below you can find an example grub.cfg that allows dual booting of Windows and Linux.
It's important to specify a timeout value above 0 if you want this modification to work.
Change the values appropriately. For example if you plan to install GRUB on the device itself, change the root to hd0,x. Also change the root partition of the linux file system accordingly if necessary.

If you install GRUB on the device and plan to dual boot with Windows, you also need to rename the /EFI/Microsoft/Boot/bootmgfw.efi file to something else and then change the name accordingly in the config. Otherwise GRUB will be skipped.

```
insmod font

if loadfont ${prefix}/unicode.pf2
then
    insmod gfxterm
    set gfxmode=auto
    set gfxpayload=keep
    terminal_output gfxterm
	set timeout=5
	menuentry "My Linux Distribution" {
        set root=(hd1,2)
        linux /zImage root=/dev/mmcblk0p6 cpuidle.off=1 console=ttyS0
        devicetree /tegra30-microsoft-surface-rt-efi.dtb
    }
	menuentry "Windows Bootloader" {
        set root=(hd1,2)
        chainloader (${root})/EFI/Microsoft/Boot/bootmgfw.efi
    }
else
    reboot
fi
```

## Build instructions:

0. git clone https://www.github.com/coherixmatts/grub-2.04
1. cd into <grub-2.04> directory
2. ./bootstrap
3. ./configure --with-platform=efi --target=arm-linux-gnueabihf
4. make
5. cd <grub-2.04>/grub-core
6. grub-mkimage -O arm-efi -d . -o grub.efi -p / part_gpt part_msdos ntfs ntfscomp hfsplus fat ext2 normal chain boot configfile linux gfxterm videoinfo efi_gop all_video video video_fb loadenv help reboot raid6rec raid5rec mdraid1x mdraid09 lvm diskfilter zfsinfo zfscrypt gcry_rijndael gcry_sha1 zfs true test sleep search search_fs_uuid search_fs_file search_label png password_pbkdf2 gcry_sha512 pbkdf2 part_apple minicmd memdisk lsacpi lssal lsefisystab lsefimmap lsefi disk keystatus jpeg iso9660 halt gfxterm_background gfxmenu trig bitmap_scale video_colors bitmap font fshelp efifwsetup echo terminal gettext efinet net priority_queue datetime bufio cat btrfs gzio lzopio crypto acpi extcmd mmap
7. it's very fast - the compiled grub.efi should now be in the <grub-core> directory


NOTES:

These instructions were tested on an Ubuntu 14.04 x64 system.  They are not 100% step-by-step instructions. Some modules and dependencies are missing. The error messages in terminal will make clear what you need to apt-get install.
