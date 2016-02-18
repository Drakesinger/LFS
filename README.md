# LFS
Linux From Scratch

## Log

### Prerequisites
Problem with git.
<pre>
WARNING: gnome-keyring:: couldn't connect to: /home/[user_name]/.cache/keyring-[generated_string]/pkcs11
</pre>
Solutions found @ [Debian Forums](https://lists.debian.org/debian-user/2014/05/msg00070.html) & [Solved Programming Problems](http://hongouru.blogspot.ch/2012/07/solved-warning-gnome-keyring-couldnt.html)


### Part I - Introduction
#### Chapter I
Just reading, nothing of note.
### Part II - Preparing for the Build
#### Chapter II - Preparing a new Partition
Problem : only have 1 partition and it's the bootable one.
Solution for VM : add a new harddrive of 12GB
HardDrive is located in /dev/sdb (port 2 SATA)
add the 10240MB partition with cfdisk /dev/sdb
and the swap partition for the remaining 2GBs

##### Make file system:

In order to see the number of the partition one must run: 
<code>
fdisk -l
</code>
   
Then using the IDs of the partitions:
<code>
mkfs -v -t ext4 /dev/sdb1
mkswap /dev/sdb5
</code>

After the filesystem has been created:
<code>
export LFS=/home/hmu/lfs
</code>

Added the export within the .bashrc and /root/.bashrc -> otherwise the env variable is 
forgotten when leaving the terminal.

Note: [What's the difference between .bashrc, .bash_profile, and .environment?](http://stackoverflow.com/questions/415403/whats-the-difference-between-bashrc-bash-profile-and-environment)
Note: [on ubuntu](https://help.ubuntu.com/community/EnvironmentVariables)

Also enabled colored terminal since was already here.
<code>
force_color_prompt=yes
</code>

##### Mounting the partition:
<code>
    mkdir -pv $LFS // Why put the -p flag? -> check doc and write here
    mount -v -t ext4 /dev/sdb1 $LFS
</code>
#### Chapter III - Packages and Patches