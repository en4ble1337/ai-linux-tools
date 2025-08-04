```bash
# Find your disk to expand
lsblk
```

```bash
# Example
node@node:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0 63.9M  1 loop /snap/core20/2318
loop1    7:1    0 38.8M  1 loop /snap/snapd/21759
loop2    7:2    0   87M  1 loop /snap/lxd/29351
sda      8:0    0  3.5T  0 disk 
├─sda1   8:1    0    1G  0 part /boot/efi
└─sda2   8:2    0 32.8G  0 part /
sr0     11:0    1 1024M  0 rom  
```

```bash
# Install the necessary utility
sudo apt update
sudo apt install cloud-guest-utils -y

# Use growpart to expand the PARTITION.
# The command is: growpart <disk> <partition_number>
sudo growpart /dev/sda 2

# Now, resize the FILESYSTEM within the partition to use the new space.
sudo resize2fs /dev/sda2
```
