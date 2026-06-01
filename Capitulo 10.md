                                                                                        Linux From Scratch - Version 12.4


Chapter 10. Making the LFS System Bootable
10.1. Introduction
 It is time to make the LFS system bootable. This chapter discusses creating the /etc/fstab file, building a kernel for
 the new LFS system, and installing the GRUB boot loader so that the LFS system can be selected for booting at startup.

10.2. Creating the /etc/fstab File
 The /etc/fstab file is used by some programs to determine where file systems are to be mounted by default, in which
 order, and which must be checked (for integrity errors) prior to mounting. Create a new file systems table like this:
 cat > /etc/fstab << "EOF"
 # Begin /etc/fstab

 # file system    mount-point       type       options                dump   fsck
 #                                                                           order

 /dev/<xxx>       /              <fff>    defaults            1              1
 /dev/<yyy>       swap           swap     pri=1               0              0
 proc             /proc          proc     nosuid,noexec,nodev 0              0
 sysfs            /sys           sysfs    nosuid,noexec,nodev 0              0
 devpts           /dev/pts       devpts   gid=5,mode=620      0              0
 tmpfs            /run           tmpfs    defaults            0              0
 devtmpfs         /dev           devtmpfs mode=0755,nosuid    0              0
 tmpfs            /dev/shm       tmpfs    nosuid,nodev        0              0
 cgroup2          /sys/fs/cgroup cgroup2 nosuid,noexec,nodev 0               0

 # End /etc/fstab
 EOF

 Replace <xxx>, <yyy>, and <fff> with the values appropriate for the system, for example, sda2, sda5, and ext4. For
 details on the six fields in this file, see fstab(5).

 Filesystems with MS-DOS or Windows origin (i.e. vfat, ntfs, smbfs, cifs, iso9660, udf) need a special option, utf8, in
 order for non-ASCII characters in file names to be interpreted properly. For non-UTF-8 locales, the value of iocharset
 should be set to be the same as the character set of the locale, adjusted in such a way that the kernel understands it. This
 works if the relevant character set definition (found under File systems -> Native Language Support when configuring
 the kernel) has been compiled into the kernel or built as a module. However, if the character set of the locale is UTF-8,
 the corresponding option iocharset=utf8 would make the file system case sensitive. To fix this, use the special option
 utf8 instead of iocharset=utf8, for UTF-8 locales. The “codepage” option is also needed for vfat and smbfs filesystems.
 It should be set to the codepage number used under MS-DOS in your country. For example, in order to mount USB
 flash drives, a ru_RU.KOI8-R user would need the following in the options portion of its mount line in /etc/fstab:
 noauto,user,quiet,showexec,codepage=866,iocharset=koi8r

 The corresponding options fragment for ru_RU.UTF-8 users is:
 noauto,user,quiet,showexec,codepage=866,utf8

 Note that using iocharset is the default for iso8859-1 (which keeps the file system case insensitive), and the utf8 option
 tells the kernel to convert the file names using UTF-8 so they can be interpreted in the UTF-8 locale.


                                                          262

                                                                                  Linux From Scratch - Version 12.4

It is also possible to specify default codepage and iocharset values for some filesystems during kernel configuration.
The relevant parameters are named “Default NLS Option” (CONFIG_NLS_DEFAULT), “Default Remote NLS Option”
(CONFIG_SMB_NLS_DEFAULT), “Default codepage for FAT” (CONFIG_FAT_DEFAULT_CODEPAGE), and “Default iocharset for
FAT” (CONFIG_FAT_DEFAULT_IOCHARSET). There is no way to specify these settings for the ntfs filesystem at kernel
compilation time.




                                                       263

                                                                                       Linux From Scratch - Version 12.4

10.3. Linux-6.16.1
 The Linux package contains the Linux kernel.
 Approximate build time: 0.4 - 32 SBU (typically about 2.5 SBU)
 Required disk space:        1.7 - 14 GB (typically about 2.3 GB)

10.3.1. Installation of the kernel
 Building the kernel involves a few steps—configuration, compilation, and installation. Read the README file in the kernel
 source tree for alternative methods to the way this book configures the kernel.

         Important
         Building the linux kernel for the first time is one of the most challenging tasks in LFS. Getting it right depends
         on the specific hardware for the target system and your specific needs. There are almost 12,000 configuration
         items that are available for the kernel although only about a third of them are needed for most computers. The
         LFS editors recommend that users not familiar with this process follow the procedures below fairly closely.
         The objective is to get an initial system to a point where you can log in at the command line when you reboot
         later in Section 11.3, “Rebooting the System.” At this point optimization and customization is not a goal.
         For general information on kernel configuration see https://www.linuxfromscratch.org/hints/downloads/files/
         kernel-configuration.txt. Additional information about configuring and building the kernel can be found at
         https://anduin.linuxfromscratch.org/LFS/kernel-nutshell/. These references are a bit dated, but still give a
         reasonable overview of the process.
         If all else fails, you can ask for help on the lfs-support mailing list. Note that subscribing is required in order
         for the list to avoid spam.

 Prepare for compilation by running the following command:
 make mrproper

 This ensures that the kernel tree is absolutely clean. The kernel team recommends that this command be issued prior to
 each kernel compilation. Do not rely on the source tree being clean after un-tarring.
 There are several ways to configure the kernel options. Usually, this is done through a menu-driven interface, for
 example:
 make menuconfig

 The meaning of optional make environment variables:
   LANG=<host_LANG_value> LC_ALL=
    This establishes the locale setting to the one used on the host. This may be needed for a proper menuconfig ncurses
    interface line drawing on a UTF-8 linux text console.
    If used, be sure to replace <host_LANG_value> by the value of the $LANG variable from your host. You can
    alternatively use instead the host's value of $LC_ALL or $LC_CTYPE.
   make menuconfig
    This launches an ncurses menu-driven interface. For other (graphical) interfaces, type make help.

         Note
         A good starting place for setting up the kernel configuration is to run make defconfig. This will set the base
         configuration to a good state that takes your current system architecture into account.


                                                          264

                                                                           Linux From Scratch - Version 12.4

Be sure to enable/disable/set the following features or the system might not work correctly or boot at all:
General setup --->
  [ ] Compile the kernel with warnings as errors                        [WERROR]
  CPU/Task time and stats accounting --->
    [*] Pressure stall information tracking                                [PSI]
    [ ]   Require boot parameter to enable pressure stall information tracking
                                                     ... [PSI_DEFAULT_DISABLED]
  < > Enable kernel headers through /sys/kernel/kheaders.tar.xz      [IKHEADERS]
  [*] Control Group support --->                                       [CGROUPS]
    [*] Memory controller                                                [MEMCG]
  [ ] Configure standard kernel features (expert users) --->            [EXPERT]

Processor type and features --->
  [*] Build a relocatable kernel                                            [RELOCATABLE]
  [*]   Randomize the address of the kernel image (KASLR)                [RANDOMIZE_BASE]

General architecture-dependent options --->
  [*] Stack Protector buffer overflow detection                         [STACKPROTECTOR]
  [*]   Strong Stack Protector                                   [STACKPROTECTOR_STRONG]

Device Drivers --->
  Generic Driver Options --->
    [ ] Support for uevent helper                                [UEVENT_HELPER]
    [*] Maintain a devtmpfs filesystem to mount at /dev                [DEVTMPFS]
    [*]   Automount devtmpfs at /dev, after the kernel mounted the rootfs
                                                           ... [DEVTMPFS_MOUNT]
  Firmware Drivers --->
    [*] Mark VGA/VBE/EFI FB as generic system framebuffer       [SYSFB_SIMPLEFB]
  Graphics support --->
    <*>    Direct Rendering Manager (XFree86 4.1.0 and higher DRI support) --->
                                                                       ... [DRM]
    [*]    Display a user-friendly message when a kernel panic occurs
                                                                ... [DRM_PANIC]
    (kmsg)   Panic screen formatter                           [DRM_PANIC_SCREEN]
    Supported DRM clients --->
      [*] Enable legacy fbdev support for your modesetting driver
                                                      ... [DRM_FBDEV_EMULATION]
    Drivers for system framebuffers --->
      <*> Simple framebuffer driver                               [DRM_SIMPLEDRM]
    Console display driver support --->
      [*] Framebuffer Console support                      [FRAMEBUFFER_CONSOLE]

Enable some additional features if you are building a 64-bit system. If you are using menuconfig, enable them
in the order of CONFIG_PCI_MSI first, then CONFIG_IRQ_REMAP, at last CONFIG_X86_X2APIC because an option only
shows up after its dependencies are selected.
Processor type and features --->
  [*] x2APIC interrupt controller architecture support                        [X86_X2APIC]

Device Drivers --->
  [*] PCI support --->                                                              [PCI]
    [*] Message Signaled Interrupts (MSI and MSI-X)                             [PCI_MSI]
  [*] IOMMU Hardware Support --->                                         [IOMMU_SUPPORT]
    [*] Support for Interrupt Remapping                                       [IRQ_REMAP]




                                               265

                                                                                     Linux From Scratch - Version 12.4

         If the partition for the LFS system is in a NVME SSD (i. e. the device node for the partition is /dev/nvme*
         instead of /dev/sd*), enable NVME support or the LFS system won't boot:
         Device Drivers --->
           NVME Support --->
             <*> NVM Express block device                                            [BLK_DEV_NVME]

There are several other options that may be desired depending on the requirements for the system. For a list of options
needed for BLFS packages, see the BLFS Index of Kernel Settings.

         Note
         If your host hardware is using UEFI and you wish to boot the LFS system with it, you should adjust some
         kernel configuration following the BLFS page even if you'll use the UEFI bootloader from the host distro.

The rationale for the above configuration items:
  Randomize the address of the kernel image (KASLR)
     Enable ASLR for kernel image, to mitigate some attacks based on fixed addresses of sensitive data or code in
     the kernel.
  Compile the kernel with warnings as errors
     This may cause building failure if the compiler and/or configuration are different from those of the kernel
     developers.
  Enable kernel headers through /sys/kernel/kheaders.tar.xz
     This will require cpio building the kernel. cpio is not installed by LFS.
  Configure standard kernel features (expert users)
     This will make some options show up in the configuration interface but changing those options may be dangerous.
     Do not use this unless you know what you are doing.
  Strong Stack Protector
     Enable SSP for the kernel. We've enabled it for the entire userspace with --enable-default-ssp configuring GCC,
     but the kernel does not use GCC default setting for SSP. We enable it explicitly here.
  Support for uevent helper
     Having this option set may interfere with device management when using Udev.
  Maintain a devtmpfs
     This will create automated device nodes which are populated by the kernel, even without Udev running. Udev
     then runs on top of this, managing permissions and adding symlinks. This configuration item is required for all
     users of Udev.
  Automount devtmpfs at /dev
     This will mount the kernel view of the devices on /dev upon switching to root filesystem just before starting init.
  Display a user-friendly message when a kernel panic occurs
     This will make the kernel correctly display the message in case a kernel panic happens and a running DRM driver
     supports to do so. Without this, it would be more difficult to diagnose a panic: if no DRM driver is running, we'd be
     on the VGA console which can only hold 24 lines and the relevant kernel message is often flushed away; if a DRM
     driver is running, the display is often completely messed up on panic. As of Linux-6.12, none of the dedicated
     drivers for mainstream GPU models really supports this, but it's supported by the “Simple framebuffer driver”
     which runs on the VESA (or EFI) framebuffer before the dedicated GPU driver is loaded. If the dedicated GPU
     driver is built as a module (instead of a part of the kernel image) and no initramfs is used, this functionality will
     work just fine before the root file system is mounted and it's already enough for providing information about most


                                                        266

                                                                                       Linux From Scratch - Version 12.4

       LFS configuration errors causing a panic (for example, an incorrect root= setting in Section 10.4, “Using GRUB
       to Set Up the Boot Process”).
  Panic screen formatter
       Set this kmsg to make sure the last kernel messages lines are displayed when a kernel panic happens. The default,
       user, would make the kernel show only a “user friendly” panic message which is not helpful on diagnostic. The
       third choice, qr_code, would make the kernel to compress the last kernel message lines into a QR code and display
       it. The QR code can hold more message lines than plain text and it can be decoded with an external device (like
       a smart phone). But it requires a Rust compiler that LFS does not provide.
  Mark VGA/VBE/EFI FB as generic system framebuffer        and Simple framebuffer driver
       These allow to use the VESA framebuffer (or the EFI framebuffer if booting the LFS system via UEFI) as a
       DRM device. The VESA framebuffer will be set up by GRUB (or the EFI framebuffer will be set up by the UEFI
       firmware), so the DRM panic handler can function before the GPU-specific DRM driver is loaded.
  Enable legacy fbdev support for your modesetting driver          and Framebuffer Console support
       These are needed to display the Linux console on a GPU driven by a DRI (Direct Rendering Infrastructure) driver.
       As CONFIG_DRM (Direct Rendering Manager) is enabled, we should enable these two options as well or we'll see
       a blank screen once the DRI driver is loaded.
  Support x2apic
       Support running the interrupt controller of 64-bit x86 processors in x2APIC mode. x2APIC may be enabled by
       firmware on 64-bit x86 systems, and a kernel without this option enabled will panic on boot if x2APIC is enabled
       by firmware. This option has no effect, but also does no harm if x2APIC is disabled by the firmware.
Alternatively, make oldconfig may be more appropriate in some situations. See the README file for more information.
If desired, skip kernel configuration by copying the kernel config file, .config, from the host system (assuming it is
available) to the unpacked linux-6.16.1 directory. However, we do not recommend this option. It is often better to
explore all the configuration menus and create the kernel configuration from scratch.
Compile the kernel image and modules:
make

If using kernel modules, module configuration in /etc/modprobe.d may be required. Information pertaining to modules
and kernel configuration is located in Section 9.3, “Overview of Device and Module Handling” and in the kernel
documentation in the linux-6.16.1/Documentation directory. Also, modprobe.d(5) may be of interest.
Unless module support has been disabled in the kernel configuration, install the modules with:
make modules_install

After kernel compilation is complete, additional steps are required to complete the installation. Some files need to be
copied to the /boot directory.

          Caution
          If you've decided to use a separate /boot partition for the LFS system (maybe sharing a /boot partition with
          the host distro), the files copied below should go there. The easiest way to do that is to create the entry for /
          boot in /etc/fstab first (read the previous section for details), then issue the following command as the root
          user in the chroot environment:
          mount /boot

          The path to the device node is omitted in the command because mount can read it from /etc/fstab.


                                                          267

                                                                                      Linux From Scratch - Version 12.4

The path to the kernel image may vary depending on the platform being used. The filename below can be changed to
suit your taste, but the stem of the filename should be vmlinuz to be compatible with the automatic setup of the boot
process described in the next section. The following command assumes an x86 architecture:
cp -iv arch/x86/boot/bzImage /boot/vmlinuz-6.16.1-lfs-12.4

System.map is a symbol file for the kernel. It maps the function entry points of every function in the kernel API, as well
as the addresses of the kernel data structures for the running kernel. It is used as a resource when investigating kernel
problems. Issue the following command to install the map file:
cp -iv System.map /boot/System.map-6.16.1

The kernel configuration file .config produced by the make menuconfig step above contains all the configuration
selections for the kernel that was just compiled. It is a good idea to keep this file for future reference:
cp -iv .config /boot/config-6.16.1

Install the documentation for the Linux kernel:
cp -r Documentation -T /usr/share/doc/linux-6.16.1

It is important to note that the files in the kernel source directory are not owned by root. Whenever a package is unpacked
as user root (like we did inside chroot), the files have the user and group IDs of whatever they were on the packager's
computer. This is usually not a problem for any other package to be installed because the source tree is removed after
the installation. However, the Linux source tree is often retained for a long time. Because of this, there is a chance
that whatever user ID the packager used will be assigned to somebody on the machine. That person would then have
write access to the kernel source.

         Note
         In many cases, the configuration of the kernel will need to be updated for packages that will be installed later
         in BLFS. Unlike other packages, it is not necessary to remove the kernel source tree after the newly built
         kernel is installed.

         If the kernel source tree is going to be retained, run chown -R 0:0 on the linux-6.16.1 directory to ensure
         all files are owned by user root.

         If you are updating the configuration and rebuilding the kernel from a retained kernel source tree, normally
         you should not run the make mrproper command. The command would purge the .config file and all the .
         o files from the previous build. Despite it's easy to restore .config from the copy in /boot, purging all the .o
         files is still a waste: for a simple configuration change, often only a few .o files need to be (re)built and the
         kernel build system will correctly skip other .o files if they are not purged.

         On the other hand, if you've upgraded GCC, you should run make clean to purge all the .o files from the
         previous build, or the new build may fail.


         Warning
         Some kernel documentation recommends creating a symlink from /usr/src/linux pointing to the kernel
         source directory. This is specific to kernels prior to the 2.6 series and must not be created on an LFS system
         as it can cause problems for packages you may wish to build once your base LFS system is complete.

                                                         268

                                                                                      Linux From Scratch - Version 12.4

10.3.2. Configuring Linux Module Load Order
 Most of the time Linux modules are loaded automatically, but sometimes it needs some specific direction. The program
 that loads modules, modprobe or insmod, uses /etc/modprobe.d/usb.conf for this purpose. This file needs to be created
 so that if the USB drivers (ehci_hcd, ohci_hcd and uhci_hcd) have been built as modules, they will be loaded in the
 correct order; ehci_hcd needs to be loaded prior to ohci_hcd and uhci_hcd in order to avoid a warning being output
 at boot time.
 Create a new file /etc/modprobe.d/usb.conf by running the following:
 install -v -m755 -d /etc/modprobe.d
 cat > /etc/modprobe.d/usb.conf << "EOF"
 # Begin /etc/modprobe.d/usb.conf

 install ohci_hcd /sbin/modprobe ehci_hcd ; /sbin/modprobe -i ohci_hcd ; true
 install uhci_hcd /sbin/modprobe ehci_hcd ; /sbin/modprobe -i uhci_hcd ; true

 # End /etc/modprobe.d/usb.conf
 EOF


10.3.3. Contents of Linux
 Installed files:             config-6.16.1, vmlinuz-6.16.1-lfs-12.4, and System.map-6.16.1
 Installed directories:       /lib/modules, /usr/share/doc/linux-6.16.1

Short Descriptions
 config-6.16.1                       Contains all the configuration selections for the kernel
 vmlinuz-6.16.1-lfs-12.4             The engine of the Linux system. When turning on the computer, the kernel is
                                     the first part of the operating system that gets loaded. It detects and initializes all
                                     components of the computer's hardware, then makes these components available
                                     as a tree of files to the software and turns a single CPU into a multitasking machine
                                     capable of running scores of programs seemingly at the same time
 System.map-6.16.1                   A list of addresses and symbols; it maps the entry points and addresses of all the
                                     functions and data structures in the kernel




                                                        269

                                                                                        Linux From Scratch - Version 12.4

10.4. Using GRUB to Set Up the Boot Process
          Note
          If your system has UEFI support and you wish to boot LFS with UEFI, you should skip the instructions in
          this page but still learn the syntax of grub.cfg and the method to specify a partition in the file from this page,
          and configure GRUB with UEFI support using the instructions provided in the BLFS page.

10.4.1. Introduction
          Warning
          Configuring GRUB incorrectly can render your system inoperable without an alternate boot device such as a
          CD-ROM or bootable USB drive. This section is not required to boot your LFS system. You may just want
          to modify your current boot loader, e.g. Grub-Legacy, GRUB2, or LILO.

 Ensure that an emergency boot disk is ready to “rescue” the computer if the computer becomes unusable (un-bootable).
 If you do not already have a boot device, you can create one. In order for the procedure below to work, you need to
 jump ahead to BLFS and install xorriso from the libisoburn package.
 cd /tmp
 grub-mkrescue --output=grub-img.iso
 xorriso -as cdrecord -v dev=/dev/cdrw blank=as_needed grub-img.iso


10.4.2. GRUB Naming Conventions
 GRUB uses its own naming structure for drives and partitions in the form of (hdn,m), where n is the hard drive number
 and m is the partition number. The hard drive numbers start from zero, but the partition numbers start from one for
 normal partitions (from five for extended partitions). Note that this is different from earlier versions where both numbers
 started from zero. For example, partition sda1 is (hd0,1) to GRUB and sdb3 is (hd1,3). In contrast to Linux, GRUB
 does not consider CD-ROM drives to be hard drives. For example, if using a CD on hdb and a second hard drive on
 hdc, that second hard drive would still be (hd1).


10.4.3. Setting Up the Configuration
 GRUB works by writing data to the first physical track of the hard disk. This area is not part of any file system. The
 programs there access GRUB modules in the boot partition. The default location is /boot/grub/.
 The location of the boot partition is a choice of the user that affects the configuration. One recommendation is to have
 a separate small (suggested size is 200 MB) partition just for boot information. That way each build, whether LFS or
 some commercial distro, can access the same boot files and access can be made from any booted system. If you choose
 to do this, you will need to mount the separate partition, move all files in the current /boot directory (e.g. the Linux
 kernel you just built in the previous section) to the new partition. You will then need to unmount the partition and
 remount it as /boot. If you do this, be sure to update /etc/fstab.
 Leaving /boot on the current LFS partition will also work, but configuration for multiple systems is more difficult.
 Using the above information, determine the appropriate designator for the root partition (or boot partition, if a separate
 one is used). For the following example, it is assumed that the root (or separate boot) partition is sda2.
 Install the GRUB files into /boot/grub and set up the boot track:

                                                          270

                                                                                    Linux From Scratch - Version 12.4


         Warning
         The following command will overwrite the current boot loader. Do not run the command if this is not desired,
         for example, if using a third party boot manager to manage the Master Boot Record (MBR).

 grub-install /dev/sda


         Note
         If the system has been booted using UEFI, grub-install will try to install files for the x86_64-efi target, but
         those files have not been installed in Chapter 8. If this is the case, add --target i386-pc to the command
         above.


10.4.4. Creating the GRUB Configuration File
 Generate /boot/grub/grub.cfg:
 cat > /boot/grub/grub.cfg << "EOF"
 # Begin /boot/grub/grub.cfg
 set default=0
 set timeout=5

 insmod part_gpt
 insmod ext2
 set root=(hd0,2)
 set gfxpayload=1024x768x32

 menuentry "GNU/Linux, Linux 6.16.1-lfs-12.4" {
         linux   /boot/vmlinuz-6.16.1-lfs-12.4 root=/dev/sda2 ro
 }
 EOF

 The insmod commands load the GRUB modules named part_gpt and ext2. Despite the naming, ext2 actually supports
 ext2, ext3, and ext4 filesystems. The grub-install command has embedded some modules into the main GRUB image
 (installed into the MBR or the GRUB BIOS partition) to access the other modules (in /boot/grub/i386-pc) without a
 chicken-or-egg issue, so with a typical configuration these two modules are already embedded and those two insmod
 commands will do nothing. But they do no harm anyway, and they may be needed with some rare configurations.

 The set gfxpayload=1024x768x32 command sets the resolution and color depth of the VESA framebuffer to be passed
 to the kernel. It's necessary for the kernel SimpleDRM driver to use the VESA framebuffer. You can use a different
 resolution or color depth value which better suits for your monitor.

         Note
         From GRUB's perspective, the kernel files are relative to the partition used. If you used a separate /boot
         partition, remove /boot from the above linux line. You will also need to change the set root line to point to
         the boot partition.




                                                        271

                                                                                   Linux From Scratch - Version 12.4


        Note
        The GRUB designator for a partition may change if you added or removed some disks (including
        removable disks like USB thumb devices). The change may cause boot failure because grub.cfg refers
        to some “old” designators. If you wish to avoid such a problem, you may use the UUID of a partition
        and the UUID of a filesystem instead of a GRUB designator to specify a device. Run lsblk -o
        UUID,PARTUUID,PATH,MOUNTPOINT to show the UUIDs of your filesystems (in the UUID column)
        and partitions (in the PARTUUID column). Then replace set root=(hdx,y) with search --set=root --fs-
        uuid <UUID of the filesystem where the kernel is installed>, and replace root=/dev/sda2 with
        root=PARTUUID=<UUID of the partition where LFS is built>.

        Note that the UUID of a partition is completely different from the UUID of the filesystem in this
        partition. Some online resources may instruct you to use root=UUID=<filesystem UUID> instead of
        root=PARTUUID=<partition UUID>, but doing so will require an initramfs, which is beyond the scope of LFS.

        The name of the device node for a partition in /dev may also change (this is less likely than a GRUB designator
        change). You can also replace paths to device nodes like /dev/sda1 with PARTUUID=<partition UUID>, in /
        etc/fstab, to avoid a potential boot failure in case the device node name has changed.


GRUB is an extremely powerful program and it provides a tremendous number of options for booting from a wide
variety of devices, operating systems, and partition types. There are also many options for customization such as
graphical splash screens, playing sounds, mouse input, etc. The details of these options are beyond the scope of this
introduction.

        Caution
        There is a command, grub-mkconfig, that can write a configuration file automatically. It uses a set of scripts
        in /etc/grub.d/ and will destroy any customizations that you make. These scripts are designed primarily for
        non-source distributions and are not recommended for LFS. If you install a commercial Linux distribution,
        there is a good chance that this program will be run. Be sure to back up your grub.cfg file.




                                                       272

