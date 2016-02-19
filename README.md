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
export LFS=/home/hmu/lfs
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

