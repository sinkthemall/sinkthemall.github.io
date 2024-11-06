+++
date = '2024-11-06T10:13:42+07:00'
draft = false
tags = ["Arch"]
title = 'How to install Arch OS'
# featured_image = 'feature_thumbnail.png'
+++

## 1. Why Arch

On a beautiful day, a programmer was fiddling with Windows theme configurations, trying to make his computer "cooler than ever." But, alas, the more he customized, the messier it became; from colors to icons, everything started to feel more "outdated" than modern. While he was holding his head in frustration due to unresolvable bugs, the universe decided to "throw" an Arch Linux distro at him, as if saying, "Try this, my child."

At first, the programmer felt overwhelmed with the chaos in front of him. But after a short time of "struggling," he discovered the truth: "Simplicity is the ultimate sophistication." Instead of trying to beautify his setup with complex themes, he could now customize a powerful operating system according to his desires, without worrying about unnecessary troubles.

That’s the beginning of my story of moving to Arch (though that’s not really how it happened :)))

So why Arch instead of Ubuntu or any other Linux distro? Here are the reasons:

- First, Arch follows the KISS principle (Keep It Simple Stupid), focusing on minimalism and keeping everything as simple as possible. Arch provides a basic setup, and from there, users configure settings to their liking. Arch is more suitable for users with long-term Linux experience who want to customize everything precisely.
- **Arch Linux** uses a rolling release model, meaning updates are done continuously without the need for major upgrades. This can lead to a more up-to-date system but requires users to be cautious with updates and comfortable with handling potential issues from ongoing updates.
- **Arch Linux** uses **`pacman`** as its package manager, known for its speed and simplicity. It also has the Arch User Repository (AUR), which contains thousands of community-maintained packages not available in the official repository. This allows users to easily install almost any software but requires some knowledge of building packages from source and security checks.
- My friend installed Arch, so I learned to install it too :))))

![Untitled](/posts/arch-install/feature.png)

(I have no prejudice against other OSes, I still use WSL daily; it’s just that Arch offers more things to learn.)

Those are the reasons why I think Arch is better than other Linux distros. Of course, for those who are lazy and don’t like diving deep into customization, Ubuntu is enough. However, the trade-off is losing control over your desired settings, and it takes up quite a bit of space. One of the things I love about Arch is its customization capability. If you search for "unixporn," a well-known Reddit community focused on customization, you’ll find thousands of visually pleasing interfaces. Additionally, installing packages on Arch is much simpler than on Ubuntu… Enough talking about its greatness, let’s start with a guide on how to install Arch. This guide will be quite long and targeted at those who have little to no experience in installing Arch, so feel free to skip sections if you’re already familiar.

Here is an overview of the guide sections:

- Basic Knowledge
    - The computer boot-up process
    - Boot modes: UEFI vs. BIOS
    - File systems
    - Partitioning
    - Mounting
    - Package manager
- Preparation
- Installing Arch

## 2. Installation

### 2.1 Basic Knowledge

#### 2.1.1 Computer Boot-Up Process

The diagram below shows the OS boot-up process for a computer:

![Untitled](/posts/arch-install/img/Untitled_1.png)

The image includes some redundant parts we don’t need to focus on. Here, we’ll only pay attention to step 4 in the diagram; steps 5 onward are automatically handled by the kernel (they only fail if there’s a configuration error or an issue with the kernel image, etc.). Below, I’ll briefly explain each step:

- Step 1 - Power On: This is the process that occurs right after pressing the power button to start the computer, during which the BIOS/UEFI initiates and performs pre-boot routines before loading the bootloader.
- Step 2 - BIOS/UEFI startup: This is the stage where BIOS/UEFI (firmware stored in ROM) performs tasks like hardware checks (Power On Self Test - POST procedure) and loading data from non-volatile memory (such as boot entries...).
- Step 3 - Detect Devices: At this point, the POST process checks hardware like RAM and hard drives, or detects boot entries from boot devices (such as hard drives, CDs, USB drives…). After checking, control is handed over to choose the boot device.
- Step 4 - Choose a Boot Device: Choosing a boot device, as a computer can run multiple operating systems and can boot from memory devices to install an OS.
- Step 5 - Run bootloader: Running the bootloader stored in the boot entry on the drive; the bootloader’s job is to load the operating system (or, more precisely, the kernel image) based on saved configurations in the boot entry → The operating system loads into memory with arguments set according to the configuration file.

At this point, the remaining tasks are handed over to the OS, and we don’t need to do much (if an error occurs during this process, it’s likely due to an issue with the kernel image). I’ve only briefly mentioned the tasks in each step. Next, I’ll explain the definitions and functions of each component in the diagram.

### 2.1.2 Boot Modes: BIOS vs UEFI

For an OS to load into memory, a boot program is essential. This is where BIOS/UEFI comes in. I think most of you who have tinkered with the BIOS menu at startup already understand this to some extent. BIOS stands for "Basic Input/Output System." Essentially, BIOS is a set of instructions stored on a firmware chip located on the computer’s motherboard.

![Untitled](/posts/arch-install/img/Untitled_2.png)

The function of BIOS has been mentioned above (hardware check, running bootloader). Replacing the outdated BIOS firmware is UEFI (Unified Extensible Firmware Interface). UEFI is an improved and upgraded version of BIOS and has almost completely replaced BIOS on computers in recent years. The reason is that BIOS limitations have hampered some newer hardware.

![Untitled](/posts/arch-install/img/Untitled_3.png)

One thing to emphasize is that, even though BIOS/UEFI is the first component to run, it is NOT an OS or anything close to an OS. Since it’s stored in ROM (to avoid potential damage and ensure smooth booting), BIOS/UEFI is much smaller than a kernel image. You can find more information on BIOS or UEFI [here](https://www.freecodecamp.org/news/uefi-vs-bios/).

### 2.1.3 Partitioning

Have you ever wondered why, with only one hard drive, your computer has two drive regions (C:\ and D:\)? This is due to partitioning. Partitioning is dividing the hard drive into separate, independent regions. Doing so creates multiple partitions, each independent of the other, so writing on one partition won’t affect the others. Additionally, certain partitions are essential to support the OS boot process.

![Untitled](/posts/arch-install/img/Untitled_4.png)

For a computer to understand the disk partitions and each partition on the drive, it needs to read the partition table. This table is stored at the beginning of the hard drive. During startup, BIOS/UEFI reads the bytes at the drive’s beginning, containing information about the partition table, partitioning scheme, partitions on the drive, and how they are organized.

![Untitled](/posts/arch-install/img/Untitled_5.png)

In terms of partitioning schemes, there are currently two main types in use:

- MBR (Master Boot Record): An older scheme used by BIOS. MBR supports up to four primary partitions or three primary partitions and one extended partition containing multiple logical partitions (I’ll explain logical and extended partitions later).
- GPT (GUID Partition Table): A new partition standard used by UEFI. GPT has no partition limit like MBR, supporting up to 2TB per partition and up to 128 partitions on Windows, or more on other OSes. This will be important later as we’ll use GPT for partitioning instead of MBR.

On a side note:

- Primary partition: Primary partitions store data or are used to boot, containing OS data.
- Extended partition: This is a special partition because it doesn’t directly store data but holds logical partitions, which are similar to folders.
- Logical partition: A sub-partition of an extended partition, functioning just like a primary partition. Logical partitions exist to bypass the primary partition limit in the MBR scheme (4 primary partitions).

Other terms you may encounter include:

- ESP (EFI System Partition): A separate partition for storing bootloaders or kernel images to assist with booting; only UEFI uses it, not BIOS.
- SWAP partition: This partition can be thought of as extended RAM. When the RAM is full, the OS moves some data from RAM to this partition to free up space. When needed, data is swapped back from this partition to RAM.

### 2.1.4 File System

A file system is a way to manage and organize the storing of data and folders. A file system is crucial; without it, the computer wouldn’t know where one file ends (obviously, not at a null byte), which folder stores which files, or how the structure is arranged. Some common file systems include (marked with * indicates those used in Arch installation):

- NTFS (New Technology File System): A new file system standard for Windows, replacing the older FAT standard, with improvements in metadata, performance, security access control, etc. Since this belongs to Windows, we won’t go into depth.

![Untitled](/posts/arch-install/img/Untitled_6.png)

- * FAT (File Allocation Table): An old file system designed for smaller disks or simpler folder structures. FAT is still in use today, known for its simplicity and high compatibility. However, due to its limitations, using FAT for data storage on a drive isn’t recommended. During the Arch installation, we’ll use FAT for the ESP partition.

![Untitled](/posts/arch-install/img/Untitled_7.png)

- * Ext4: Simply put, this is the main file system for data storage on Linux, similar to NTFS for Windows. Ext4 offers features like compatibility, extents, and large file size support.

![Untitled](/posts/arch-install/img/Untitled_8.png)

- * SWAP: This is the file system for the SWAP partition.

### 2.1.5 Mounting

When mentioning file systems and partitions, another term we need to cover is "mounting." So what is mounting? Simply put, mounting makes a partition accessible. Why is this necessary? When booting, the OS has no information about the drive or its partitions and file systems. Therefore, to store and access data on the drive, mounting is required.

![Untitled](/posts/arch-install/img/Untitled_9.png)

### 2.1.6 Package Manager

Here’s ChatGPT’s explanation of a package manager:

> A package manager is a software tool that automates the process of installing, upgrading, configuring, and removing computer programs for a computer's operating system in a consistent manner. It is a critical component in software management, simplifying the process of maintaining computers and software applications.

A package manager handles installing, updating, or removing packages. If you’re on Windows, you download an installer from the web to install an application. In Arch, this is automated with the “pacman” command (or “apt” for Ubuntu).

This is an overview of the knowledge needed to install Arch. If you prefer a manual installation and already know the basics, you can skip this section. However, I think it will help you better understand and troubleshoot boot issues.

## 2.2 Preparation

- A USB drive
- Arch ISO file
- Computer
- Ethernet connection (for installation), or Wi-Fi or anything else, as long as it can connect to the internet (ping arch.org).

## 2.3 Installing Arch

This guide is based on an article about installing Arch on Hyper-V. Although it’s for a virtual machine, I think many parts are similar to installing on a physical machine. You can find the article [here](https://github.com/kourbou/how-to-hyperv-arch).

- Arch ISO file: Download directly from [archlinux.org/download](https://archlinux.org/download/), using a torrent or wget magnet link.
- Burn ISO file to USB: This doesn’t need further explanation, and I’m too lazy to write a guide. If you’re unfamiliar with this process, check this link [here](https://unetbootin.github.io/#:~:text=Using_UNetbootin,boot_from_the_USB_drive.) for reference.

![Untitled](/posts/arch-install/img/Untitled_10.png)

- Adjust boot order: Before installation, adjust the boot order to place the USB boot first. To do this, open the BIOS menu and set the USB as the first boot device:

![Untitled](/posts/arch-install/img/Untitled_11.png)

Another important note: Arch does not support secure boot, so disable it in the security settings.

- When restarting and reaching this screen, select the Arch Linux install medium.

![Untitled](/posts/arch-install/img/Untitled_12.png)

Once booted, you’ll access a shell that looks like this:

![Untitled](/posts/arch-install/img/Untitled_13.png)

First, partition the hard drive as follows:

- According to the original article, the author suggested using fdisk to partition, but I recommend against it since fdisk uses the MBR scheme. Instead, use gdisk to partition. Device files are typically located in the /dev folder, and if there’s only one hard drive (SSD), the filename representing the drive will be /dev/sda (for HDDs, it’s /dev/hda).
- Enter `gdisk <device filename>` to start partitioning; after entering, you’ll see something like this:

![Untitled](/posts/arch-install/img/Untitled_14.png)

- Follow the steps to partition the disk (here, I’m creating three different partitions: ESP, SWAP, and root partition, with partitions /dev/sda1, /dev/sda2, and /dev/sda3. ESP will be around 512MB, SWAP about 4GB, and the root partition will use the remaining space).

```flow
root@archiso ~ # gdisk /dev/sda
GPT fdisk (gdisk) version 1.0.9.1

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help): n
Partition number (1-128, default 1): ↵
First sector (34-41943006, default = 2048) or {+-}size{KMGTP}: ↵
Last sector (2048-41943006, default = 41940991) or {+-}size{KMGTP}: +512M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ↵
Changed type of partition to 'Linux filesystem'

Command (? for help): n
Partition number (2-128, default 2): ↵
First sector (34-41943006, default = 1050624) or {+-}size{KMGTP}: ↵
Last sector (1050624-41943006, default = 41940991) or {+-}size{KMGTP}: +4G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ↵
Changed type of partition to 'Linux filesystem'

Command (? for help): n
Partition number (3-128, default 3): ↵
First sector (34-41943006, default = 9439232) or {+-}size{KMGTP}: ↵
Last sector (9439232-41943006, default = 41940991) or {+-}size{KMGTP}: ↵
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): ↵
Changed type of partition to 'Linux filesystem'

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sda.
The operation has completed successfully.
```

Once completed, you need to format each partition's file system individually. As mentioned above, `/dev/sda1` (ESP) will be formatted as `FAT`, `/dev/sda2` as `swap`, and `/dev/sda3` as `ext4`. I’ll format `ESP` as `FAT32` since FAT supports backward compatibility.

```flow
root@archiso ~ # mkfs.fat -F 32 /dev/sda1
mkfs.fat 4.2 (2021-01-31)
root@archiso ~ # mkswap /dev/sda2
Setting up swapspace version 1, size = 4 GiB (4294963200 bytes)
no label, UUID=373190da-e759-46ab-bc2f-3a5f87d3b159
root@archiso ~ # swapon /dev/sda2
root@archiso ~ # mkfs.ext4 /dev/sda3
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done
Creating filesystem with 4062720 4k blocks and 1015808 inodes
Filesystem UUID: d49e04ef-1353-497d-8458-a4d784969e88
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

Now the partitioning is complete, so the next step is mounting them to access each partition for installation:

```flow
root@archiso ~ # mount /dev/sda3 /mnt
root@archiso ~ # mkdir /mnt/boot
root@archiso ~ # mount /dev/sda1 /mnt/boot
root@archiso ~ #
```

Here, I have mounted `/dev/sda3` (root partition) to the `/mnt` directory and `/dev/sda1` (ESP) to `/mnt/boot`. The `/mnt/boot` directory will store the kernel image and necessary components for booting.

Next, we’ll install the core components of Arch OS. This installation can be done using `pacstrap`—similar to `pacman`, but used for installing into a specific folder. Required packages include: `base linux linux-firmware dhcpcd openssh neovim`.

```flow
root@archiso ~ # pacstrap /mnt base linux linux-firmware dhcpcd openssh neovim
```

The installation process might take some time, so feel free to grab a snack or some tea while waiting (though it’s actually pretty quick, about 1-2 minutes).

![Untitled](/posts/arch-install/img/Untitled_15.png)

![Untitled](/posts/arch-install/img/Untitled_16.png)

```flow
==> WARNING: Possibly missing firmware for module: 'qla1280'
==> WARNING: Possibly missing firmware for module: 'wd719x'
  -> Running build hook: [filesystems]
  -> Running build hook: [fsck]
zstd: error 70 : Write error : cannot write block : No space left on device
==> Generating module dependencies
cp: error writing '/tmp/mkinitcpio.2XehQT/root/lib/modules/6.7.4-arch1-1/modules.builtin': No space left on device
cp: error writing '/tmp/mkinitcpio.2XehQT/root/lib/modules/6.7.4-arch1-1/modules.builtin.modinfo': No space left on device
cp: error writing '/tmp/mkinitcpio.2XehQT/root/lib/modules/6.7.4-arch1-1/modules.order': No space left on device
depmod: ERROR: Could not create index 'modules.dep'. Output is truncated: No space left on device
==> Creating zstd-compressed initcpio image: '/boot/initramfs-linux-fallback.img'
==> WARNING: errors were encountered during the build. The image may not be complete.
error: command failed to execute correctly
(13/13) Reloading system bus configuration...
  Skipped: Running in chroot.
pacstrap /mnt base linux linux-firmware dhcpcd openssh neovim  31.20s user 24.81s system 44% cpu 2:07.24 total
root@archiso ~ #
```

Next, generate a file system table for each partition. When the system boots, it will look for a /mnt/etc/fstab file and mount partitions based on this information. Therefore, we’ll use genfstab to generate and save the fs table to the root partition.

```flow
root@archiso ~ # genfstab -U /mnt >> /mnt/etc/fstab
root@archiso ~ # cat /mnt/etc/fstab
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
# /dev/sda3
UUID=d49e04ef-1353-497d-8458-a4d784969e88       /               ext4            rw,relatime     0 1

# /dev/sda1
UUID=0A43-B4DE          /boot           vfat            rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro       0 2

# /dev/sda2
UUID=373190da-e759-46ab-bc2f-3a5f87d3b159       none            swap            defaults        0 0

root@archiso ~ #
```

Now, I’ll switch to the root to set up a few configurations:

```flow
root@archiso ~ # arch-chroot /mnt
```

- Setting local timezone:

```flow
[root@archiso /]# ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
[root@archiso /]# date
Sat Feb 17 02:28:10 +07 2024
```

- Run `locale-gen`; first, open `/etc/locale.gen` and uncomment the line `en_US.UTF-8 UTF-8`

```flow
[root@archiso /]# nvim /etc/locale.gen
```

![Untitled](/posts/arch-install/img/Untitled_17.png)

```flow
[root@archiso /]# locale-gen
Generating locales...
  en_US.UTF-8... done
Generation complete.
[root@archiso /]#
```

- Create `/etc/locale.conf` and set the `LANG`

```flow
[root@archiso /]# nvim /etc/locale.conf
LANG=en_US.UTF-8
```

- Setup keyboard map:

```flow
[root@archiso /]# nvim /etc/vconsole.conf
KEYMAP=us
```

- Setup hostname và hosts

```flow
[root@archiso /]# nvim /etc/hostname
4rk-zer0
[root@archiso /]# nvim /etc/hosts
127.0.0.1	localhost
::1		localhost

```
- Enable some services like DHCP and SSH:

```flow
[root@archiso /]# systemctl enable dhcpcd.service
Created symlink /etc/systemd/system/multi-user.target.wants/dhcpcd.service → /usr/lib/systemd/system/dhcpcd.service.
[root@archiso /]# systemctl enable sshd.service
Created symlink /etc/systemd/system/multi-user.target.wants/sshd.service → /usr/lib/systemd/system/sshd.service.
[root@archiso /]#
```
The final step is to set up the bootloader. A quick note: the Linux kernel has a configuration called `CONFIG_EFI_STUB`, which allows the kernel to boot directly from EFI (or UEFI) without needing a bootloader. I’ll cover bootloader installation in a separate section later. For now, let’s proceed with the setup.

- Exit the current chroot environment by pressing Ctrl + D.

![Untitled](/posts/arch-install/img/Untitled_18.png)

- Find the UUID of the root partition using the `blkid` command:

```flow
root@archiso ~ # blkid /dev/sda3
/dev/sda3: UUID="d49e04ef-1353-497d-8458-a4d784969e88" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="Linux filesystem" PARTUUID="84a09143-1874-4a04-82b7-fa9ef3d12b4b"
root@archiso ~ #
```

- Create a boot entry using `efibootmgr`, replacing the PARTUUID in the command with the PARTUUID of the root partition:

```flow
root@archiso ~ # efibootmgr --disk /dev/sda --part 1 --create --label "Linux Kernel" --loader /vmlinuz-linux --verbose -
-unicode 'root=PARTUUID=84a09143-1874-4a04-82b7-fa9ef3d12b4b rw initrd=\initramfs-linux.img'
BootCurrent: 0002
Timeout: 0 seconds
BootOrder: 0003,0002,0000,0001
Boot0000* EFI Network   AcpiEx(VMBus,,)/VenHw(9b17e5a2-0891-42dd-b653-80b5c22809ba,635161f83edfc546913ff2d2f965ed0e36a4db2a250997498f05925300e1f7c6)/MAC(000000000000,0)/IPv4(0.0.0.00.0.0.0,0,0)
      dp: 02 02 18 00 00 00 00 00 00 00 00 00 00 00 00 00 56 4d 42 75 73 00 00 00 / 01 04 34 00 a2 e5 17 9b 91 08 dd 42 b6 53 80 b5 c2 28 09 ba 63 51 61 f8 3e df c5 46 91 3f f2 d2 f9 65 ed 0e 36 a4 db 2a 25 09 97 49 8f 05 92 53 00 e1 f7 c6 / 03 0b 25 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 / 03 0c 1b 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 / 7f ff 04 00
Boot0001* EFI SCSI Device       AcpiEx(VMBus,,)/VenHw(9b17e5a2-0891-42dd-b653-80b5c22809ba,d96361baa104294db60572e2ffb1dc7f60966f49d23c65408ca66f4766cadb78)/SCSI(0,0)
      dp: 02 02 18 00 00 00 00 00 00 00 00 00 00 00 00 00 56 4d 42 75 73 00 00 00 / 01 04 34 00 a2 e5 17 9b 91 08 dd 42 b6 53 80 b5 c2 28 09 ba d9 63 61 ba a1 04 29 4d b6 05 72 e2 ff b1 dc 7f 60 96 6f 49 d2 3c 65 40 8c a6 6f 47 66 ca db 78 / 03 02 08 00 00 00 00 00 / 7f ff 04 00
Boot0002* EFI SCSI Device       AcpiEx(VMBus,,)/VenHw(9b17e5a2-0891-42dd-b653-80b5c22809ba,d96361baa104294db60572e2ffb1dc7f60966f49d23c65408ca66f4766cadb78)/SCSI(0,1)
      dp: 02 02 18 00 00 00 00 00 00 00 00 00 00 00 00 00 56 4d 42 75 73 00 00 00 / 01 04 34 00 a2 e5 17 9b 91 08 dd 42 b6 53 80 b5 c2 28 09 ba d9 63 61 ba a1 04 29 4d b6 05 72 e2 ff b1 dc 7f 60 96 6f 49 d2 3c 65 40 8c a6 6f 47 66 ca db 78 / 03 02 08 00 00 00 01 00 / 7f ff 04 00
Boot0003* Linux Kernel  HD(1,GPT,69b4003a-1656-4d4d-8f53-5024facf44d5,0x800,0x100000)/File(\vmlinuz-linux)root=PARTUUID=84a09143-1874-4a04-82b7-fa9ef3d12b4b rw initrd=\initramfs-linux.img
      dp: 04 01 2a 00 01 00 00 00 00 08 00 00 00 00 00 00 00 00 10 00 00 00 00 00 3a 00 b4 69 56 16 4d 4d 8f 53 50 24 fa cf 44 d5 02 02 / 04 04 22 00 5c 00 76 00 6d 00 6c 00 69 00 6e 00 75 00 7a 00 2d 00 6c 00 69 00 6e 00 75 00 78 00 00 00 / 7f ff 04 00
    data: 72 00 6f 00 6f 00 74 00 3d 00 50 00 41 00 52 00 54 00 55 00 55 00 49 00 44 00 3d 00 38 00 34 00 61 00 30 00 39 00 31 00 34 00 33 00 2d 00 31 00 38 00 37 00 34 00 2d 00 34 00 61 00 30 00 34 00 2d 00 38 00 32 00 62 00 37 00 2d 00 66 00 61 00 39 00 65 00 66 00 33 00 64 00 31 00 32 00 62 00 34 00 62 00 20 00 72 00 77 00 20 00 69 00 6e 00 69 00 74 00 72 00 64 00 3d 00 5c 00 69 00 6e 00 69 00 74 00 72 00 61 00 6d 00 66 00 73 00 2d 00 6c 00 69 00 6e 00 75 00 78 00 2e 00 69 00 6d 00 67 00
root@archiso ~ #
```

- Check if the boot entry exists:

![Untitled](/posts/arch-install/img/Untitled_19.png)

Basically, we're almost done here, but there’s one more task left: creating a user.

```flow
root@archiso ~ # arch-chroot /mnt
[root@archiso /]# pacman -Syu sudo
[root@archiso /]# useradd -m d4rkn19ht
[root@archiso /]# usermod -aG wheel d4rkn19ht
[root@archiso /]# passwd d4rkn19ht
New password:
Retype new password:
passwd: password updated successfully
[root@archiso /]#
```

Change the super user permissions in the sudo command by uncommenting the line `wheel ALL=(ALL) ALL`.

```flow
# EDITOR=nvim visudo
...
%wheel ALL=(ALL) ALL
```

Press `Ctrl + D` to exit the chroot environment, then power off, adjust the boot order, and reboot the computer. If the screen appears as shown below upon reboot, it means the setup was successful.

![Untitled](/posts/arch-install/img/Untitled_20.png)

Some notes:

- If there are boot issues (fstab error, etc.), you can reinsert the USB boot drive and use the shell to fix any necessary settings. Remember to mount before making changes.
- Typically, if there's an error, it’s likely due to an fstab issue or a boot entry problem. In this case, don't rush to reinstall everything from scratch; simply recreate the fstab and generate a new boot entry (make sure to delete the old one).
- Installing Arch is fast, but it doesn’t include a GUI by default. I’ll write a separate post about setting up a GUI.

That concludes this guide. See you in another post :P
