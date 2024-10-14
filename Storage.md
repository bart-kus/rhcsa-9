# Storage

## General notes

### Deleting partitions
Deleting partitions in Fdisk does not delete the data on the partition. The only thing that's deleted is a line in the partition table. If you delete a partition in order to extend it, when you create a new one, Fdisk asks "*Do you want to remove the signature? [Y]es/[N]o"*
Select **no**, otherwise you will delete all data on the partition.

### Listing devices
To see disks and partitions. ``lsblk``
If for some reason you don't think that is correct you can ``cat /proc/partitions``

## Mount

This command checks the "/etc/fstab" file is valid.
``findmnt --verify`` 

To mount all unmounted devices.
``mount -a``

*During the exam, reboot the machine to verify all mounts!*
If your system can't boot because of a problem with "/etc/fstab" you will fail the exam.

In datacenter environments, block device names may change. Different solutions exist for persistent naming.

- **UUID**: a UUID is automatically generated for each device that contains a filesystem or anything similar.
- **Label**: while creating the filesystem, the option ``-L`` can be used to set an arbitrary name that can be used for mounting the filesystem.

To set a label on an XFS filesystem you can use ``xfs_admin -L mygreatlabel /dev/sda6`` To label it the volume needs to be unmounted. In "/etc/fstab" the **first field** would be ``LABEL=mygreatlabel`` instead of "/dev/sda6."

If you want to mount it based on UUID, you can find the UUID with ``blkid``.

### Mount via Systemd

Based on the example below, the mount filename would be mnt-vda6.mount and located in /etc/systemd/system/. \
\
[Unit]\
Description=Mount vda6 \
\
[Mount] \
What=UUID="07b6fb67-8687-40d5-81a4-bb58e28e1e0a" \
Where=/mnt/vda6 \
Type=xfs \
Options=defaults \
\
[Install] \
WantedBy=multi-user.target

## Swap

### Swap from partition

1. Create the parition. Partitions types are important on the exam. In Fdisk remember to set the type to swap.
2. Set up a Linux swap area on the device or file we created. ``mkswap /dev/vdd1`` You could also do ``mkswap -L myswapspace /dev/vdd1`` and use the label to mount it in fstab.
3. Mount the swap space in "/etc/fstab". In the first field you can use /dev/vdd1 or the UUID, or the label if you created one. ![mkswap](pictures/mkswap.png)
4. In the "/etc/fstab" file remember to mount the swap file to **none** and set the filesystem as swap.
5. To activate the swap run ``swapon -a`` 
6. Check out the new swap space ``free -h``

If you have multiple swap files you can see the priority with ``swapon -s``

### Swap from file

1. sudo dd if=/dev/zero of=/swap count=2048 bs=1MiB
2. chmod 600 /swap
3. mkswap /swap
4. swapon /swap
5. fstab: /swap swap swap defaults 0 0

## To create a new partition in RHEL 9, you can use the fdisk or parted tools. Here's how you can use fdisk to create a new partition on a disk.

#### Steps to Create a New Partition in RHEL 9:
1. List Available Disks:
First, find the disk on which you want to create the partition.


``lsblk``
This will list all the block devices on the system. Identify the disk you want to partition, such as /dev/sda, /dev/nvme0n1, or another device name.

2. Open the Disk with fdisk:
Use fdisk to modify the partition table of the disk.


``sudo fdisk /dev/sdX``
Replace /dev/sdX with the appropriate disk, such as /dev/sda (do not specify a partition like /dev/sda1, use the whole disk).

3. Enter fdisk Command Mode:
When you run fdisk, you'll see a command prompt with options to create, delete, and manage partitions. Some important commands are:

- m: Show help with available commands.
- p: Print the partition table (to view existing partitions).
- n: Create a new partition.
- d: Delete a partition.
- w: Write changes and exit.
- q: Quit without saving changes.
4. Create a New Partition:
To create a new partition:

Press n to create a new partition.
Select the partition type:
p for primary partition (usually up to 4 primary partitions).
e for an extended partition.
Choose the partition number (usually defaults to the next available number).
Specify the first sector (press Enter to use the default, which is recommended).
Specify the last sector or size of the partition (e.g., +10G to create a 10 GB partition).
Example interaction:


Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-20971519, default 2048): (press Enter)
Last sector, +sectors or +size{K,M,G,T,P} (2048-20971519, default 20971519): +10G

5. Write the Changes:
After defining the partition, you need to save the changes to the disk:


Command (m for help): w
This writes the partition table changes and exits fdisk.

6. Update Kernel Partition Table:
To inform the system about the partition table changes, you can either reboot the system or run:


``sudo partprobe /dev/sdX``
This will reload the partition table without needing a reboot.

7. Format the New Partition:
Now that the partition is created, format it with a file system (e.g., ext4).


``sudo mkfs.ext4 /dev/sdX1``
Replace /dev/sdX1 with your newly created partition (like /dev/sda1).

8. Mount the Partition:
To mount the new partition, create a mount point and mount it.

Create a directory to mount the partition:


``sudo mkdir /mnt/new_partition``
Mount the partition:


``sudo mount /dev/sdX1 /mnt/new_partition``
9. Make the Mount Permanent (Optional):
To ensure the partition mounts automatically after reboot, add it to the /etc/fstab file.

Get the UUID of the partition:


``blkid /dev/sdX1``  -- command to see blkid (UUID)
Edit the /etc/fstab file and add an entry like this:


UUID=<partition-uuid>   /mnt/new_partition   ext4   defaults   0 0
Replace <partition-uuid> with the actual UUID, and set the mount point and file system type accordingly.

Example of Full Command Workflow:
bash
Copy code
lsblk                                # List available disks
sudo fdisk /dev/sda                  # Open the disk with fdisk
n                                    # Create a new partition
p                                    # Choose primary partition
+10G                                 # Set the size to 10GB
w                                    # Write changes to the partition table
sudo partprobe /dev/sda              # Update kernel partition table
sudo mkfs.ext4 /dev/sda1             # Format the new partition with ext4
sudo mkdir /mnt/new_partition        # Create a mount point
sudo mount /dev/sda1 /mnt/new_partition  # Mount the new partition
