# LFS
Linux From Scratch Project

# Work Log

# Prerequisites
In order to share files between systems, the git program was used.

Problem with git.
```bash
WARNING: gnome-keyring:: couldn't connect to: /home/[user_name]/.cache/keyring-[generated_string]/pkcs11
```
Solutions found @ [Debian Forums](https://lists.debian.org/debian-user/2014/05/msg00070.html) & [Solved Programming Problems](http://hongouru.blogspot.ch/2012/07/solved-warning-gnome-keyring-couldnt.html)


# Part I - Introduction

## Chapter I
Just reading, nothing of note.

# Part II - Preparing for the Build

## Chapter II - Preparing a new Partition
In this chapter we need to create a new partition where the LFS Linux kernel will be located.

The problem : only have 1 partition and it's the bootable one.

Solution for VM : add a new harddrive of 12GB

HardDrive is located in /dev/sdb (port 2 SATA)
- add the 10240MB partition with
```bash
cfdisk /dev/sdb
```
- and the swap partition for the remaining 2GBs

Solution for Ubuntu Laptop :

Reboot the machine with the Ubuntu USB stick inserted. Boot on the Live Ubuntu Image and get gparted. Resize to heart's content and follow the rest of the instructions (or just do everything with gparted).

[Source](http://askubuntu.com/questions/291888/can-i-adjust-reduce-my-partition-size-for-ubuntu)

### Make file system:

In order to see the number of the partition one must run:

```bash
fdisk -l
```

Then using the IDs of the partitions:
```bash
mkfs -v -t ext4 /dev/sdb1
mkswap /dev/sdb5
```

After the filesystem has been created:
```bash
export LFS=/mnt/lfs
```

Added the export within the .bashrc and /root/.bashrc -> otherwise the env variable is
forgotten when leaving the terminal.

Note: [What's the difference between .bashrc, .bash_profile, and .environment?](http://stackoverflow.com/questions/415403/whats-the-difference-between-bashrc-bash-profile-and-environment)

Note: [on ubuntu](https://help.ubuntu.com/community/EnvironmentVariables)

Also enabled colored terminal since was already here.
```bash
force_color_prompt=yes
```

##### Mounting the partition:
`mkdir -pv $LFS` Why put the -p flag? -> check doc and write here <br>
`mount -v -t ext4 /dev/sdb1 $LFS` Mount the partition
#### Chapter III - Packages and Patches
Downloaded all sources by using the wget-list.txt provided by the LFS author/team.
#### Chapter IV - Final Preparations
As root, create the tools directory and a symbolic link to it on the host system.
```bash
sudo mkdir -v $LFS/tools
sudo ln -sv $LFS/tools /
```
TODO : Write what was done after
#### Chapter V - Constructing a Temporary System
Q: What is a toolchain?

A: Compiler + Assembler + Linker + Libraries

Config.guess: i686-pc-linux-gnu

Dynamic linker: ld-linux.so.2

Command :
```bash
$ readelf -l <binary_file_name> | grep interpreter
[Requesting program interpreter: /lib/ld-linux.so.2]
```
# Part III - Building the LFS System
#### Chapter VI

* Entering the Chroot environment
```bash
chroot "$LFS" /tools/bin/env -i \
HOME=/root \
TERM="$TERM" \
PS1='\u:\w\$ ' \
PATH=/bin:/usr/bin:/sbin:/usr/sbin:/tools/bin \
/tools/bin/bash --login +h
```


* Locales:
```bash
localedef -i fr_CH -f UTF-8 fr_CH.UTF-8
localedef -i fr_CH -f ISO-8859-1 fr_CH
```

Notes:

Don't forget to run the tests on ACL.
Kept the sources, delete after.

* Shadow
    * left <code>GROUP=999</code> by default
    * Changed CREATE_MAIL_SPOOL=no
        using <code>sed -i 's/yes/no/' /etc/default/useradd</code>

##### Testsuites not run:

* Libtool-2.4
* Inetutils-1.9.4
* Perl
* Autoconf
* Automake
* Diffutils
* GetText
* Gperf
* GROFF

    Configured with the following:
    ```bash
    PAGE=A4 ./configure --prefix=/usr
    ```

* XZ-5.2.1

	Problem after installation.
	```bash
	ln -svf ../../lib/$(readlink /usr/lib/liblzma.so) /usr/lib/liblzma.so
	```
	However, after some study (and using <code> ls -lFahH --color </code>):

	The symbolic link /usr/lib/liblzma.so will point to

	```bash
	lrwxrwxrwx  1 root root    26 Mar  7 16:29 liblzma.so -> ../../lib/liblzma.so.5.2.1*
	```

	/lib is the root, may have been easier to point directly to /lib insted of ../../lib.
    This is made in case the user does not exist?

	Looks like acctually all symbolic links to libraries not in the user use the same type of paths.

* Kmod Problem

    Last symbolic link could not be created. Tried to remake the package -> failure. had to restore back to a previous image.
    After rebuilding and modifying everything from the Autoconf tool, everything works.

* Eudev

    Command to update hardware device info:
    ```bash
    LD_LIBRARY_PATH=/tools/lib udevadm hwdb --update
    ```
    Needs to be run after every hardware update

#### Chapter 7 - Configuration

In order to share stuff between the debian machine and everything else I set up git.
The IP address of the virtual machine was wrong and had to renew the lease.
Done with:
```bash
$ sudo dhclient -r eth0
$ sudo dhclient eth0
```

Configuring the linux console, keymap setup is a bit more complicated since on debian and the man pages for
dumpkeys / keymaps / loadkeys all point to a location that is not valid on a debian system.

File usr/share/connsole-setup/console-setup content:
```bash
CHARMAP=guess

CODESET=guess
FONTFACE=TerminusBold
FONTSIZE=16
```

After trying to read the documentation and getting a headache. Honestly, this is one of the worst documentation I have ever seen.
Trying to retrieve the charset of the current console, the font used, the keymaps is simply stupid and un-doable.
Configured the console as such:

```bash
cat > /etc/sysconfig/console << "EOF"
# Begin /etc/sysconfig/console
UNICODE="1"
KEYMAP="qwertz/fr_CH"
FONT="lat2-16 -m 8859-15"
# End /etc/sysconfig/console
EOF
```
Notes

Font: no idea, cannot determine from the Terminus font readme which should be used and the [Francophone HOW-TO](http://www.tldp.org/HOWTO/Francophones-HOWTO-5.html#ss5.4)
is simply impossible to read.

sysconfig/rc.site configuration
```bash
DISTRO="LFS-10-Mars-2016" # The distro name
DISTRO_CONTACT="horiamut@msn.com" # Bug report address
DISTRO_MINI="LFS-10-Mars-2016" # Short name used in filenames for distro config

[...]

# Write out fsck progress if yes
VERBOSE_FSCK=yes
```
```bash
root:/etc# LC_ALL=fr_CH.iso88591 locale charmap
ISO-8859-1
root:/etc# LC_ALL=fr_CH.iso88591 locale language
French
root:/etc# LC_ALL=fr_CH.iso88591 locale int_curr_symbol
CHF
root:/etc# LC_ALL=fr_CH.iso88591 locale int_prefix     
41
root:/etc#
```

```bash
cat > /etc/profile << "EOF"
# Begin /etc/profile
export LANG=fr_CH.ISO-8859-1
# End /etc/profile
EOF
```

FSTAB
```bash
cat > /etc/fstab << "EOF"
# Begin /etc/fstab
# file system / mount-point / filesystem type / options / dump / fsck
# order
/dev/sdb1   /               ext4            defaults                1   1
/dev/sdb5   swap            swap            pri=1                   0   0
proc        /proc           proc            nosuid,noexec,nodev     0   0
sysfs       /sys            sysfs           nosuid,noexec,nodev     0   0
devpts      /dev/pts        devpts          gid=5,mode=620          0   0
tmpfs       /run            tmpfs           defaults                0   0
devtmpfs    /dev            devtmpfs        mode=0755,nosuid        0   0
/dev/sr0    /media/cdrom0   udf,iso9660     user,noauto             0   0
# End /etc/fstab
EOF
```
```bash
make mrproper
make defconfig

make menuconfig
CONFIG_LOCALVERSION="LFS-Horia-Mut-10-Mar-16"

make
```

GRUB Config
----------------------
```bash
grub-install /dev/sda
```

```bash
cat > /boot/grub/grub.cfg << "EOF"
# Begin /boot/grub/grub.cfg
set default=0
set timeout=5
insmod ext2
set root=(hd1,1)
menuentry "GNU/Linux, Linux 4.2-lfs-7.8 by Mut Horia" {
linux /boot/vmlinuz-4.2-lfs-7.8 root=/dev/sdb1 ro
}
EOF
```

THE End
-------------------------
```bash
cat > /etc/lsb-release << "EOF"
DISTRIB_ID="Linux From Scratch"
DISTRIB_RELEASE="7.8"
DISTRIB_CODENAME="LFS by Horia Mut"
DISTRIB_DESCRIPTION="Linux From Scratch"
EOF
```
