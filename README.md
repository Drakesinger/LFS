# LFS
Linux From Scratch

## Log

### Prerequisites
Problem with git.
```bash
WARNING: gnome-keyring:: couldn't connect to: /home/[user_name]/.cache/keyring-[generated_string]/pkcs11
```
Solutions found @ [Debian Forums](https://lists.debian.org/debian-user/2014/05/msg00070.html) & [Solved Programming Problems](http://hongouru.blogspot.ch/2012/07/solved-warning-gnome-keyring-couldnt.html)


### Part I - Introduction

#### Chapter I
Just reading, nothing of note.

### Part II - Preparing for the Build

#### Chapter II - Preparing a new Partition
Problem : only have 1 partition and it's the bootable one.

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

##### Make file system:

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

#### Chapter VI

Entering the Chroot environment
```bash
chroot "$LFS" /tools/bin/env -i \
HOME=/root \
TERM="$TERM" \
PS1='\u:\w\$ ' \
PATH=/bin:/usr/bin:/sbin:/usr/sbin:/tools/bin \
/tools/bin/bash --login +h
```


Locales:
```bash
localedef -i fr_CH -f UTF-8 fr_CH.UTF-8
localedef -i fr_CH -f ISO-8859-1 fr_CH
```

Notes: Don't forget to run the tests on ACL.
Kept the sources, delete after.

Shadow: left <code>GROUP=999</code> by default
Changed CREATE_MAIL_SPOOL=no
using <code>sed -i 's/yes/no/' /etc/default/useradd</code>

#### Not Tested:

* Libtool-2.4 
    
    Did not launch the test suite.
    Letting the extracted sources there to be tested after automake is installed

* Inetutils-1.9.4
        
    Not running tests. Run at end.

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
Looks weird. However, after some study. (and using <code> ls -lFahH --color </code>)

The symbolic link /usr/lib/liblzma.so will point to:

```bash
lrwxrwxrwx  1 root root    26 Mar  7 16:29 liblzma.so -> ../../lib/liblzma.so.5.2.1*
```

/lib is the root, may have been easier to point dirrectly to /lib insted of ../../lib.
Looks like acctually all symbolic links to libraries not in the user use the same type of paths.

* Kmod Problem

   Last symbolic link could not be created. Tried to remake the package -> failure. had to restore back to a previous image.
   After rebuilding and modifying everything from the Autoconf tool, everything works.
