                                                                                      Linux From Scratch - Version 12.4


Chapter 11. The End
11.1. The End
 Well done! The new LFS system is installed! We wish you much success with your shiny new custom-built Linux
 system.
 It may be a good idea to create an /etc/lfs-release file. By having this file, it is very easy for you (and for us if you
 need to ask for help at some point) to find out which LFS version is installed on the system. Create this file by running:
 echo 12.4 > /etc/lfs-release

 Two files describing the installed system may be used by packages that can be installed on the system later, either in
 binary form or by building them.
 The first one shows the status of your new system with respect to the Linux Standards Base (LSB). To create this
 file, run:
 cat > /etc/lsb-release << "EOF"
 DISTRIB_ID="Linux From Scratch"
 DISTRIB_RELEASE="12.4"
 DISTRIB_CODENAME="<your name here>"
 DISTRIB_DESCRIPTION="Linux From Scratch"
 EOF

 The second one contains roughly the same information, and is used by systemd and some graphical desktop
 environments. To create this file, run:
 cat > /etc/os-release << "EOF"
 NAME="Linux From Scratch"
 VERSION="12.4"
 ID=lfs
 PRETTY_NAME="Linux From Scratch 12.4"
 VERSION_CODENAME="<your name here>"
 HOME_URL="https://www.linuxfromscratch.org/lfs/"
 RELEASE_TYPE="stable"
 EOF

 Be sure to customize the fields 'DISTRIB_CODENAME' and 'VERSION_CODENAME' to make the system uniquely
 yours.

11.2. Get Counted
 Now that you have finished the book, do you want to be counted as an LFS user? Head over to https://www.
 linuxfromscratch.org/cgi-bin/lfscounter.php and register as an LFS user by entering your name and the first LFS version
 you have used.
 Let's reboot into LFS now.

11.3. Rebooting the System
 Now that all of the software has been installed, it is time to reboot your computer. However, there are still a few things
 to check. Here are some suggestions:
 • Install any firmware needed if the kernel driver for your hardware requires some firmware files to function
   properly.

                                                         273

                                                                                      Linux From Scratch - Version 12.4

 • Ensure a password is set for the root user.
 • A review of the following configuration files is also appropriate at this point.
   • /etc/fstab
   • /etc/hosts
   • /etc/inputrc
   • /etc/profile
   • /etc/resolv.conf
   • /etc/vimrc
   • /etc/sysconfig/ifconfig.eth0
 Now that we have said that, let's move on to booting our shiny new LFS installation for the first time! First exit from
 the chroot environment:
 logout

 Then unmount the virtual file systems:
 umount -v $LFS/dev/pts
 mountpoint -q $LFS/dev/shm && umount -v $LFS/dev/shm
 umount -v $LFS/dev
 umount -v $LFS/run
 umount -v $LFS/proc
 umount -v $LFS/sys

 If multiple partitions were created, unmount the other partitions before unmounting the main one, like this:
 umount -v $LFS/home
 umount -v $LFS

 Unmount the LFS file system itself:
 umount -v $LFS

 Now, reboot the system.
 Assuming the GRUB boot loader was set up as outlined earlier, the menu is set to boot LFS 12.4 automatically.
 When the reboot is complete, the LFS system is ready for use. What you will see is a simple “login: ” prompt. At this
 point, you can proceed to the BLFS Book where you can add more software to suit your needs.
 If your reboot is not successful, it is time to troubleshoot. For hints on solving initial booting problems, see https://
 www.linuxfromscratch.org/lfs/troubleshooting.html.

11.4. Additional Resources
 Thank you for reading this LFS book. We hope that you have found this book helpful and have learned more about
 the system creation process.
 Now that the LFS system is installed, you may be wondering “What next?” To answer that question, we have compiled
 a list of resources for you.
 • Maintenance
                                                         274

                                                                                        Linux From Scratch - Version 12.4

   Bugs and security notices are reported regularly for all software. Since an LFS system is compiled from source,
   it is up to you to keep abreast of such reports. There are several online resources that track such reports, some of
   which are shown below:
   • LFS Security Advisories
      This is a list of security vulnerabilities discovered in the LFS book after it's published.
   • Open Source Security Mailing List
      This is a mailing list for discussion of security flaws, concepts, and practices in the Open Source community.
 • LFS Hints
   The LFS Hints are a collection of educational documents submitted by volunteers in the LFS community. The
   hints are available at https://www.linuxfromscratch.org/hints/downloads/files/.
 • Mailing lists
   There are several LFS mailing lists you may subscribe to if you are in need of help, want to stay current with
   the latest developments, want to contribute to the project, and more. See Chapter 1 - Mailing Lists for more
   information.
 • The Linux Documentation Project
   The goal of The Linux Documentation Project (TLDP) is to collaborate on all of the issues of Linux
   documentation. The TLDP features a large collection of HOWTOs, guides, and man pages. It is located at https://
   tldp.org/.

11.5. Getting Started After LFS
11.5.1. Deciding what to do next
 Now that LFS is complete and you have a bootable system, what do you do? The next step is to decide how to use it.
 Generally, there are two broad categories to consider: workstation or server. Indeed, these categories are not mutually
 exclusive. The applications needed for each category can be combined onto a single system, but let's look at them
 separately for now.
 A server is the simpler category. Generally this consists of a web server such as the Apache HTTP Server and a database
 server such as MariaDB. However other services are possible. The operating system embedded in a single use device
 falls into this category.
 On the other hand, a workstation is much more complex. It generally requires a graphical user environment such as
 LXDE, XFCE, KDE, or Gnome based on a basic graphical environment and several graphical based applications such
 as the Firefox web browser, Thunderbird email client, or LibreOffice office suite. These applications require many
 (several hundred depending on desired capabilities) more packages of support applications and libraries.
 In addition to the above, there is a set of applications for system management for all kinds of systems. These applications
 are all in the BLFS book. Not all packages are needed in every environment. For example dhcpcd, is not normally
 appropriate for a server and wireless_tools, are normally only useful for a laptop system.

11.5.2. Working in a basic LFS environment
 When you initially boot into LFS, you have all the internal tools to build additional packages. Unfortunately, the user
 environment is quite sparse. There are a couple of ways to improve this:


                                                          275

                                                                                    Linux From Scratch - Version 12.4

11.5.2.1. Work from the LFS host in chroot
 This method provides a complete graphical environment where a full featured browser and copy/paste capabilities are
 available. This method allows using applications like the host's version of wget to download package sources to a
 location available when working in the chroot environment.
 In order to properly build packages in chroot, you will also need to remember to mount the virtual file systems if they
 are not already mounted. One way to do this is to create a script on the HOST system:
 cat > ~/mount-virt.sh << "EOF"
 #!/bin/bash

 function mountbind
 {
    if ! mountpoint $LFS/$1 >/dev/null; then
      $SUDO mount --bind /$1 $LFS/$1
      echo $LFS/$1 mounted
    else
      echo $LFS/$1 already mounted
    fi
 }

 function mounttype
 {
    if ! mountpoint $LFS/$1 >/dev/null; then
      $SUDO mount -t $2 $3 $4 $5 $LFS/$1
      echo $LFS/$1 mounted
    else
      echo $LFS/$1 already mounted
    fi
 }

 if [ $EUID -ne 0 ]; then
    SUDO=sudo
 else
    SUDO=""
 fi

 if [ x$LFS == x ]; then
    echo "LFS not set"
    exit 1
 fi

 mountbind dev
 mounttype dev/pts devpts devpts -o gid=5,mode=620
 mounttype proc     proc   proc
 mounttype sys      sysfs sysfs
 mounttype run      tmpfs run
 if [ -h $LFS/dev/shm ]; then
    install -v -d -m 1777 $LFS$(realpath /dev/shm)
 else
    mounttype dev/shm tmpfs tmpfs -o nosuid,nodev
 fi

 #mountbind usr/src
 #mountbind boot
 #mountbind home
 EOF

 Note that the last three commands in the script are commented out. These are useful if those directories are mounted as
 separate partitions on the host system and will be mounted when booting the completed LFS/BLFS system.

                                                        276

                                                                                        Linux From Scratch - Version 12.4

 The script can be run with bash ~/mount-virt.sh as either a regular user (recommended) or as root. If run as a regular
 user, sudo is required on the host system.
 Another issue pointed out by the script is where to store downloaded package files. This location is arbitrary. It can
 be in a regular user's home directory such as ~/sources or in a global location like /usr/src. Our recommendation is not
 to mix BLFS sources and LFS sources in (from the chroot environment) /sources. In any case, the packages must be
 accessible inside the chroot environment.
 A last convenience feature presented here is to streamline the process of entering the chroot environment. This can be
 done with an alias placed in a user's ~/.bashrc file on the host system:
 alias lfs='sudo /usr/sbin/chroot /mnt/lfs /usr/bin/env -i HOME=/root TERM="$TERM" PS1="\u:\w\\\\$ "
 PATH=/usr/bin:/usr/sbin /bin/bash --login'

 This alias is a little tricky because of the quoting and levels of backslash characters. It must be all on a single line. The
 above command has been split in two for presentation purposes.

11.5.2.2. Work remotely via ssh
 This method also provides a full graphical environment, but first requires installing sshd on the LFS system, usually
 in chroot. It also requires a second computer. This method has the advantage of being simple by not requiring the
 complexity of the chroot environment. It also uses your LFS built kernel for all additional packages and still provides
 a complete system for installing packages.
 You may use the scp command to upload the package sources to be built onto the LFS system. If you want to download
 the sources onto the LFS system directly instead, install libtasn1, p11-kit, make-ca, and wget in chroot (or upload their
 sources using scp after booting the LFS system).

11.5.2.3. Work from the LFS command line
 This method requires installing libtasn1, p11-kit, make-ca, wget, gpm, and links (or lynx) in chroot and then rebooting
 into the new LFS system. At this point the default system has six virtual consoles. Switching consoles is as easy as
 using the Alt+Fx key combinations where Fx is between F1 and F6. The Alt+← and Alt+→ combinations also will
 change the console.
 At this point you can log into two different virtual consoles and run the links or lynx browser in one console and bash
 in the other. GPM then allows copying commands from the browser with the left mouse button, switching consoles,
 and pasting into the other console.

          Note
          As a side note, switching of virtual consoles can also be done from an X Window instance with the
          Ctrl+Alt+Fx key combination, but the mouse copy operation does not work between the graphical interface
          and a virtual console. You can return to the X Window display with the Ctrl+Alt+Fx combination, where
          Fx is usually F1 but may be F7.




                                                           277

