```bash
# Find your disk to expand
lsblk
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
