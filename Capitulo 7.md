                                                                                    Linux From Scratch - Version 12.4


Chapter 7. Entering Chroot and Building Additional
Temporary Tools
7.1. Introduction
 This chapter shows how to build the last missing bits of the temporary system: the tools needed to build the various
 packages. Now that all circular dependencies have been resolved, a “chroot” environment, completely isolated from
 the host operating system (except for the running kernel), can be used for the build.
 For proper operation of the isolated environment, some communication with the running kernel must be established.
 This is done via the so-called Virtual Kernel File Systems, which will be mounted before entering the chroot
 environment. You may want to verify that they are mounted by issuing the findmnt command.
 Until Section 7.4, “Entering the Chroot Environment”, the commands must be run as root, with the LFS variable set.
 After entering chroot, all commands are run as root, fortunately without access to the OS of the computer you built
 LFS on. Be careful anyway, as it is easy to destroy the whole LFS system with bad commands.

7.2. Changing Ownership
         Note
         The commands in the remainder of this book must be performed while logged in as user root and no longer
         as user lfs. Also, double check that $LFS is set in root's environment.

 Currently, the whole directory hierarchy in $LFS is owned by the user lfs, a user that exists only on the host system.
 If the directories and files under $LFS are kept as they are, they will be owned by a user ID without a corresponding
 account. This is dangerous because a user account created later could get this same user ID and would own all the files
 under $LFS, thus exposing these files to possible malicious manipulation.
 To address this issue, change the ownership of the $LFS/* directories to user root by running the following command:
 chown --from lfs -R root:root $LFS/{usr,var,etc,tools}
 case $(uname -m) in
   x86_64) chown --from lfs -R root:root $LFS/lib64 ;;
 esac


7.3. Preparing Virtual Kernel File Systems
 Applications running in userspace utilize various file systems created by the kernel to communicate with the kernel
 itself. These file systems are virtual: no disk space is used for them. The content of these file systems resides in
 memory. These file systems must be mounted in the $LFS directory tree so the applications can find them in the chroot
 environment.
 Begin by creating the directories on which these virtual file systems will be mounted:
 mkdir -pv $LFS/{dev,proc,sys,run}


7.3.1. Mounting and Populating /dev
 During a normal boot of an LFS system, the kernel automatically mounts the devtmpfs file system on the /dev directory;
 the kernel creates device nodes on that virtual file system during the boot process, or when a device is first detected
 or accessed. The udev daemon may change the ownership or permissions of the device nodes created by the kernel,

                                                         79

                                                                                       Linux From Scratch - Version 12.4

 and create new device nodes or symlinks, to ease the work of distro maintainers and system administrators. (See
 Section 9.3.2.2, “Device Node Creation” for details.) If the host kernel supports devtmpfs, we can simply mount a
 devtmpfs at $LFS/dev and rely on the kernel to populate it.

 But some host kernels lack devtmpfs support; these host distros use different methods to create the content of /dev. So
 the only host-agnostic way to populate the $LFS/dev directory is by bind mounting the host system's /dev directory. A
 bind mount is a special type of mount that makes a directory subtree or a file visible at some other location. Use the
 following command to do this.
 mount -v --bind /dev $LFS/dev


7.3.2. Mounting Virtual Kernel File Systems
 Now mount the remaining virtual kernel file systems:
 mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
 mount -vt proc proc $LFS/proc
 mount -vt sysfs sysfs $LFS/sys
 mount -vt tmpfs tmpfs $LFS/run

 The meaning of the mount options for devpts:
   gid=5
      This ensures that all devpts-created device nodes are owned by group ID 5. This is the ID we will use later on for the
      tty group. We use the group ID instead of a name, since the host system might use a different ID for its tty group.
   mode=0620
      This ensures that all devpts-created device nodes have mode 0620 (user readable and writable, group writable).
      Together with the option above, this ensures that devpts will create device nodes that meet the requirements of
      grantpt(), meaning the Glibc pt_chown helper binary (which is not installed by default) is not necessary.
 In some host systems, /dev/shm is a symbolic link to a directory, typically /run/shm. The /run tmpfs was mounted above
 so in this case only a directory needs to be created with the correct permissions.
 In other host systems /dev/shm is a mount point for a tmpfs. In that case the mount of /dev above will only create /dev/
 shm as a directory in the chroot environment. In this situation we must explicitly mount a tmpfs:
 if [ -h $LFS/dev/shm ]; then
    install -v -d -m 1777 $LFS$(realpath /dev/shm)
 else
    mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
 fi


7.4. Entering the Chroot Environment
 Now that all the packages which are required to build the rest of the needed tools are on the system, it is time to enter
 the chroot environment and finish installing the temporary tools. This environment will also be used to install the final
 system. As user root, run the following command to enter the environment that is, at the moment, populated with
 nothing but temporary tools:
 chroot "$LFS" /usr/bin/env -i   \
     HOME=/root                  \
     TERM="$TERM"                \
     PS1='(lfs chroot) \u:\w\$ ' \
     PATH=/usr/bin:/usr/sbin     \
     MAKEFLAGS="-j$(nproc)"      \
     TESTSUITEFLAGS="-j$(nproc)" \
     /bin/bash --login


                                                          80

                                                                                        Linux From Scratch - Version 12.4

 If you don't want to use all available logical cores, replace $(nproc) with the number of logical cores you want to use
 for building packages in this chapter and the following chapters. The test suites of some packages (notably Autoconf,
 Libtool, and Tar) in Chapter 8 are not affected by MAKEFLAGS, they use a TESTSUITEFLAGS environment variable instead.
 We set that here as well for running these test suites with multiple cores.
 The -i option given to the env command will clear all the variables in the chroot environment. After that, only the HOME,
 TERM, PS1, and PATH variables are set again. The TERM=$TERM construct sets the TERM variable inside chroot to the same
 value as outside chroot. This variable is needed so programs like vim and less can operate properly. If other variables
 are desired, such as CFLAGS or CXXFLAGS, this is a good place to set them.
 From this point on, there is no need to use the LFS variable any more because all work will be restricted to the LFS file
 system; the chroot command runs the Bash shell with the root (/) directory set to $LFS.

 Notice that /tools/bin is not in the PATH. This means that the cross toolchain will no longer be used.

 Also note that the bash prompt will say I have no name! This is normal because the /etc/passwd file has not been
 created yet.

          Note
          It is important that all the commands throughout the remainder of this chapter and the following chapters
          are run from within the chroot environment. If you leave this environment for any reason (rebooting for
          example), ensure that the virtual kernel filesystems are mounted as explained in Section 7.3.1, “Mounting and
          Populating /dev” and Section 7.3.2, “Mounting Virtual Kernel File Systems” and enter chroot again before
          continuing with the installation.


7.5. Creating Directories
 It is time to create the full directory structure in the LFS file system.

          Note
          Some of the directories mentioned in this section may have already been created earlier with explicit
          instructions, or when installing some packages. They are repeated below for completeness.

 Create some root-level directories that are not in the limited set required in the previous chapters by issuing the following
 command:
 mkdir -pv /{boot,home,mnt,opt,srv}




                                                            81

                                                                                         Linux From Scratch - Version 12.4

 Create the required set of subdirectories below the root-level by issuing the following commands:
 mkdir -pv /etc/{opt,sysconfig}
 mkdir -pv /lib/firmware
 mkdir -pv /media/{floppy,cdrom}
 mkdir -pv /usr/{,local/}{include,src}
 mkdir -pv /usr/lib/locale
 mkdir -pv /usr/local/{bin,lib,sbin}
 mkdir -pv /usr/{,local/}share/{color,dict,doc,info,locale,man}
 mkdir -pv /usr/{,local/}share/{misc,terminfo,zoneinfo}
 mkdir -pv /usr/{,local/}share/man/man{1..8}
 mkdir -pv /var/{cache,local,log,mail,opt,spool}
 mkdir -pv /var/lib/{color,misc,locate}

 ln -sfv /run /var/run
 ln -sfv /run/lock /var/lock

 install -dv -m 0750 /root
 install -dv -m 1777 /tmp /var/tmp

 Directories are, by default, created with permission mode 755, but this is not desirable everywhere. In the commands
 above, two changes are made—one to the home directory of user root, and another to the directories for temporary files.
 The first mode change ensures that not just anybody can enter the /root directory—just like a normal user would do
 with his or her own home directory. The second mode change makes sure that any user can write to the /tmp and /
 var/tmp directories, but cannot remove another user's files from them. The latter is prohibited by the so-called “sticky
 bit,” the highest bit (1) in the 1777 bit mask.

7.5.1. FHS Compliance Note
 This directory tree is based on the Filesystem Hierarchy Standard (FHS) (available at https://refspecs.linuxfoundation.
 org/fhs.shtml). The FHS also specifies the optional existence of additional directories such as /usr/local/games and
 /usr/share/games. In LFS, we create only the directories that are really necessary. However, feel free to create more
 directories, if you wish.

          Warning
          The FHS does not mandate the existence of the directory /usr/lib64, and the LFS editors have decided not
          to use it. For the instructions in LFS and BLFS to work correctly, it is imperative that this directory be non-
          existent. From time to time you should verify that it does not exist, because it is easy to create it inadvertently,
          and this will probably break your system.


7.6. Creating Essential Files and Symlinks
 Historically, Linux maintained a list of the mounted file systems in the file /etc/mtab. Modern kernels maintain this
 list internally and expose it to the user via the /proc filesystem. To satisfy utilities that expect to find /etc/mtab, create
 the following symbolic link:
 ln -sv /proc/self/mounts /etc/mtab

 Create a basic /etc/hosts file to be referenced in some test suites, and in one of Perl's configuration files as well:
 cat > /etc/hosts << EOF
 127.0.0.1 localhost $(hostname)
 ::1        localhost
 EOF


                                                            82

                                                                                   Linux From Scratch - Version 12.4

In order for user root to be able to login and for the name “root” to be recognized, there must be relevant entries in
the /etc/passwd and /etc/group files.

Create the /etc/passwd file by running the following command:
cat > /etc/passwd << "EOF"
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/dev/null:/usr/bin/false
daemon:x:6:6:Daemon User:/dev/null:/usr/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/run/dbus:/usr/bin/false
uuidd:x:80:80:UUID Generation Daemon User:/dev/null:/usr/bin/false
nobody:x:65534:65534:Unprivileged User:/dev/null:/usr/bin/false
EOF

The actual password for root will be set later.

Create the /etc/group file by running the following command:
cat > /etc/group << "EOF"
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tape:x:4:
tty:x:5:
daemon:x:6:
floppy:x:7:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
input:x:24:
mail:x:34:
kvm:x:61:
uuidd:x:80:
wheel:x:97:
users:x:999:
nogroup:x:65534:
EOF

The created groups are not part of any standard—they are groups decided on in part by the requirements of the Udev
configuration in Chapter 9, and in part by common conventions employed by a number of existing Linux distributions.
In addition, some test suites rely on specific users or groups. The Linux Standard Base (LSB, available at https://
refspecs.linuxfoundation.org/lsb.shtml) only recommends that, besides the group root with a Group ID (GID) of 0, a
group bin with a GID of 1 be present. The GID of 5 is widely used for the tty group, and the number 5 is also used in /
etc/fstab for the devpts filesystem. All other group names and GIDs can be chosen freely by the system administrator
since well-written programs do not depend on GID numbers, but rather use the group's name.

The ID 65534 is used by the kernel for NFS and separate user namespaces for unmapped users and groups (those exist
on the NFS server or the parent user namespace, but “do not exist” on the local machine or in the separate namespace).
We assign nobody and nogroup to avoid an unnamed ID. But other distros may treat this ID differently, so any portable
program should not depend on this assignment.


                                                        83

                                                                                    Linux From Scratch - Version 12.4

Some tests in Chapter 8 need a regular user. We add this user here and delete this account at the end of that chapter.
echo "tester:x:101:101::/home/tester:/bin/bash" >> /etc/passwd
echo "tester:x:101:" >> /etc/group
install -o tester -d /home/tester

To remove the “I have no name!” prompt, start a new shell. Since the /etc/passwd and /etc/group files have been
created, user name and group name resolution will now work:
exec /usr/bin/bash --login

The login, agetty, and init programs (and others) use a number of log files to record information such as who was
logged into the system and when. However, these programs will not write to the log files if they do not already exist.
Initialize the log files and give them proper permissions:
touch /var/log/{btmp,lastlog,faillog,wtmp}
chgrp -v utmp /var/log/lastlog
chmod -v 664 /var/log/lastlog
chmod -v 600 /var/log/btmp

The /var/log/wtmp file records all logins and logouts. The /var/log/lastlog file records when each user last logged
in. The /var/log/faillog file records failed login attempts. The /var/log/btmp file records the bad login attempts.

        Note
        The /run/utmp file records the users that are currently logged in. This file is created dynamically in the boot
        scripts.

        Note
        The utmp, wtmp, btmp, and lastlog files use 32-bit integers for timestamps and they'll be fundamentally broken
        after year 2038. Many packages have stopped using them and other packages are going to stop using them.
        It is probably best to consider them deprecated.




                                                        84

                                                                                        Linux From Scratch - Version 12.4

7.7. Gettext-0.26
 The Gettext package contains utilities for internationalization and localization. These allow programs to be compiled
 with NLS (Native Language Support), enabling them to output messages in the user's native language.
 Approximate build time:         1.5 SBU
 Required disk space:            463 MB

7.7.1. Installation of Gettext
 For our temporary set of tools, we only need to install three programs from Gettext.
 Prepare Gettext for compilation:
 ./configure --disable-shared

 The meaning of the configure option:
   --disable-shared
        We do not need to install any of the shared Gettext libraries at this time, therefore there is no need to build them.
 Compile the package:
 make

 Install the msgfmt, msgmerge, and xgettext programs:
 cp -v gettext-tools/src/{msgfmt,msgmerge,xgettext} /usr/bin

 Details on this package are located in Section 8.33.2, “Contents of Gettext.”




                                                            85

                                                                                       Linux From Scratch - Version 12.4

7.8. Bison-3.8.2
 The Bison package contains a parser generator.
 Approximate build time:         0.2 SBU
 Required disk space:            58 MB

7.8.1. Installation of Bison
 Prepare Bison for compilation:
 ./configure --prefix=/usr \
             --docdir=/usr/share/doc/bison-3.8.2

 The meaning of the new configure option:
   --docdir=/usr/share/doc/bison-3.8.2
        This tells the build system to install bison documentation into a versioned directory.
 Compile the package:
 make

 Install the package:
 make install

 Details on this package are located in Section 8.34.2, “Contents of Bison.”




                                                           86

                                                                                         Linux From Scratch - Version 12.4

7.9. Perl-5.42.0
 The Perl package contains the Practical Extraction and Report Language.
 Approximate build time:          0.6 SBU
 Required disk space:             295 MB

7.9.1. Installation of Perl
 Prepare Perl for compilation:
 sh Configure -des                                          \
              -D prefix=/usr                                \
              -D vendorprefix=/usr                          \
              -D useshrplib                                 \
              -D privlib=/usr/lib/perl5/5.42/core_perl      \
              -D archlib=/usr/lib/perl5/5.42/core_perl      \
              -D sitelib=/usr/lib/perl5/5.42/site_perl      \
              -D sitearch=/usr/lib/perl5/5.42/site_perl     \
              -D vendorlib=/usr/lib/perl5/5.42/vendor_perl \
              -D vendorarch=/usr/lib/perl5/5.42/vendor_perl

 The meaning of the Configure options:
   -des
        This is a combination of three options: -d uses defaults for all items; -e ensures completion of all tasks; -s silences
        non-essential output.
   -D vendorprefix=/usr
        This ensures perl knows how to tell packages where they should install their Perl modules.
   -D useshrplib
        Build libperl needed by some Perl modules as a shared library, instead of a static library.
   -D privlib,-D archlib,-D sitelib,...
        These settings define where Perl looks for installed modules. The LFS editors chose to put them in a directory
        structure based on the MAJOR.MINOR version of Perl (5.42) which allows upgrading Perl to newer patch levels
        (the patch level is the last dot separated part in the full version string like 5.42.0) without reinstalling all of the
        modules.
 Compile the package:
 make

 Install the package:
 make install

 Details on this package are located in Section 8.43.2, “Contents of Perl.”




                                                             87

                                                                                       Linux From Scratch - Version 12.4

7.10. Python-3.13.7
 The Python 3 package contains the Python development environment. It is useful for object-oriented programming,
 writing scripts, prototyping large programs, and developing entire applications. Python is an interpreted computer
 language.
 Approximate build time:          0.5 SBU
 Required disk space:             546 MB

7.10.1. Installation of Python
            Note
            There are two package files whose name starts with the “python” prefix. The one to extract from is Python-
            3.13.7.tar.xz (notice the uppercase first letter).


 Prepare Python for compilation:
 ./configure --prefix=/usr       \
             --enable-shared     \
             --without-ensurepip \
             --without-static-libpython

 The meaning of the configure option:
   --enable-shared
        This switch prevents installation of static libraries.
   --without-ensurepip
        This switch disables the Python package installer, which is not needed at this stage.
   --without-static-libpython
        This switch prevents building a large, but unneeded, static library.
 Compile the package:
 make


            Note
            Some Python 3 modules can't be built now because the dependencies are not installed yet. For the ssl module,
            a message Python requires a OpenSSL 1.1.1 or newer is outputted. The message should be ignored. Just
            make sure the toplevel make command has not failed. The optional modules are not needed now and they
            will be built in Chapter 8.

 Install the package:
 make install

 Details on this package are located in Section 8.51.2, “Contents of Python 3.”




                                                                 88

                                                                                 Linux From Scratch - Version 12.4

7.11. Texinfo-7.2
 The Texinfo package contains programs for reading, writing, and converting info pages.
 Approximate build time:       0.2 SBU
 Required disk space:          152 MB

7.11.1. Installation of Texinfo
 Prepare Texinfo for compilation:
 ./configure --prefix=/usr

 Compile the package:
 make

 Install the package:
 make install

 Details on this package are located in Section 8.72.2, “Contents of Texinfo.”




                                                         89

                                                                                       Linux From Scratch - Version 12.4

7.12. Util-linux-2.41.1
 The Util-linux package contains miscellaneous utility programs.
 Approximate build time:         0.2 SBU
 Required disk space:            192 MB

7.12.1. Installation of Util-linux
 The FHS recommends using the /var/lib/hwclock directory instead of the usual /etc directory as the location for the
 adjtime file. Create this directory with:

 mkdir -pv /var/lib/hwclock

 Prepare Util-linux for compilation:
 ./configure --libdir=/usr/lib     \
             --runstatedir=/run    \
             --disable-chfn-chsh   \
             --disable-login       \
             --disable-nologin     \
             --disable-su          \
             --disable-setpriv     \
             --disable-runuser     \
             --disable-pylibmount \
             --disable-static      \
             --disable-liblastlog2 \
             --without-python      \
             ADJTIME_PATH=/var/lib/hwclock/adjtime \
             --docdir=/usr/share/doc/util-linux-2.41.1

 The meaning of the configure options:
   ADJTIME_PATH=/var/lib/hwclock/adjtime
        This sets the location of the file recording information about the hardware clock in accordance to the FHS. This
        is not strictly needed for this temporary tool, but it prevents creating a file at another location, which would not
        be overwritten or removed when building the final util-linux package.
   --libdir=/usr/lib
        This switch ensures the .so symlinks targeting the shared library file in the same directory (/usr/lib) directly.
   --disable-*
        These switches prevent warnings about building components that require packages not in LFS or not installed yet.
   --without-python
        This switch disables using Python. It avoids trying to build unneeded bindings.
   runstatedir=/run
        This switch sets the location of the socket used by uuidd and libuuid correctly.
 Compile the package:
 make

 Install the package:
 make install

 Details on this package are located in Section 8.79.2, “Contents of Util-linux.”

                                                            90

                                                                                      Linux From Scratch - Version 12.4

7.13. Cleaning up and Saving the Temporary System
7.13.1. Cleaning
 First, remove the currently installed documentation files to prevent them from ending up in the final system, and to
 save about 35 MB:
 rm -rf /usr/share/{info,man,doc}/*

 Second, on a modern Linux system, the libtool .la files are only useful for libltdl. No libraries in LFS are loaded by
 libltdl, and it's known that some .la files can cause BLFS package failures. Remove those files now:
 find /usr/{lib,libexec} -name \*.la -delete

 The current system size is now about 3 GB, however the /tools directory is no longer needed. It uses about 1 GB of
 disk space. Delete it now:
 rm -rf /tools


7.13.2. Backup
 At this point the essential programs and libraries have been created and your current LFS system is in a good state.
 Your system can now be backed up for later reuse. In case of fatal failures in the subsequent chapters, it often turns out
 that removing everything and starting over (more carefully) is the best way to recover. Unfortunately, all the temporary
 files will be removed, too. To avoid spending extra time to redo something which has been done successfully, creating
 a backup of the current LFS system may prove useful.

          Note
          All the remaining steps in this section are optional. Nevertheless, as soon as you begin installing packages
          in Chapter 8, the temporary files will be overwritten. So it may be a good idea to do a backup of the current
          system as described below.

 The following steps are performed from outside the chroot environment. That means you have to leave the chroot
 environment first before continuing. The reason for that is to get access to file system locations outside of the chroot
 environment to store/read the backup archive, which ought not be placed within the $LFS hierarchy.
 If you have decided to make a backup, leave the chroot environment:
 exit


          Important
          All of the following instructions are executed by root on your host system. Take extra care about the
          commands you're going to run as mistakes made here can modify your host system. Be aware that the
          environment variable LFS is set for user lfs by default but may not be set for root.
          Whenever commands are to be executed by root, make sure you have set LFS.
          This has been discussed in Section 2.6, “Setting the $LFS Variable and the Umask.”

 Before making a backup, unmount the virtual file systems:
 mountpoint -q $LFS/dev/shm && umount $LFS/dev/shm
 umount $LFS/dev/pts
 umount $LFS/{sys,proc,run,dev}



                                                          91

                                                                                    Linux From Scratch - Version 12.4

 Make sure you have at least 1 GB free disk space (the source tarballs will be included in the backup archive) on the
 file system containing the directory where you create the backup archive.
 Note that the instructions below specify the home directory of the host system's root user, which is typically found
 on the root file system. Replace $HOME by a directory of your choice if you do not want to have the backup stored in
 root's home directory.

 Create the backup archive by running the following command:

         Note
         Because the backup archive is compressed, it takes a relatively long time (over 10 minutes) even on a
         reasonably fast system.

 cd $LFS
 tar -cJpf $HOME/lfs-temp-tools-12.4.tar.xz .


         Note
         If continuing to chapter 8, don't forget to reenter the chroot environment as explained in the “Important” box
         below.

7.13.3. Restore
 In case some mistakes have been made and you need to start over, you can use this backup to restore the system and
 save some recovery time. Since the sources are located under $LFS, they are included in the backup archive as well,
 so they do not need to be downloaded again. After checking that $LFS is set properly, you can restore the backup by
 executing the following commands:

         Warning
         The following commands are extremely dangerous. If you run rm -rf ./* as the root user and you do not
         change to the $LFS directory or the LFS environment variable is not set for the root user, it will destroy your
         entire host system. YOU ARE WARNED.

 cd $LFS
 rm -rf ./*
 tar -xpf $HOME/lfs-temp-tools-12.4.tar.xz

 Again, double check that the environment has been set up properly and continue building the rest of the system.

         Important
         If you left the chroot environment to create a backup or restart building using a restore, remember to check
         that the virtual file systems are still mounted (findmnt | grep $LFS). If they are not mounted, remount them
         now as described in Section 7.3, “Preparing Virtual Kernel File Systems” and re-enter the chroot environment
         (see Section 7.4, “Entering the Chroot Environment”) before continuing.




                                                         92

