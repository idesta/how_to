# Expanding Linux Server Disk Volume

```
This is debian based disk expanding procudere. You may exclude the SWAP Partition configuration if you want to configure other non-swap linux partitions
```

## Move Swap space to make space for root partition

- Scenario: It is required to increase the root partition size of a Debian 10 system. Here
is layout of the disk.

> **sudo fdisk -l**

- You will observe 3 partitions
    - /dev/sda1 ----------------> Type (Linux)
    - /dev/sda2 ----------------> Type (Extended)
    - /dev/sda5 ----------------> Type (Swap)

```
For example the size of the disk /dev/sda is 200 GiB (after extending).  

The three partitions use the first sectors only. If we expand the root partition (/dev/sda1);  

it will overlap with the unused partition /devsda2 and swap partition /dev/sda5.  

So we have to move the swap partition elsewhere; either use swapfile or create another partition.  

```

- **Steps to create a separate partition at the end of the disk without changing the size**

    - Turn off your current swap partition.
    - Delete partitions
    - Expand /dev/sda1
    - Create /dev/sda5 at the end and enable swap

## 1. Turn off your current swap partition.

- **Identify the swap partition**

- > sudo swapon -s

- **Turn off the current swap device**

- > sudo swapoff /dev/sda5

    - May be it will take long time to stop/off the swap partition

- **Update /etc/fstab**

- > sudo nano /etc/fstab

    - comment out the swap UUID entry. (add '#' in front of the UUID entry of swap file system table)

- **Check the swap partition is disabled**

- > sudo swapon -s

    - If there is unusual output; it will be fixed after reboot. [rebooting is not urgent at this time]

## 2. Edit the partition

- **2.1 Delete the swap and extended partition**

- **2.2 Expand/resize the root partition**

- **2.3 Create the swap and extended partition**

#### 2.1 Delete the swap and extended partition

- **Edit the partition**

- > sudo fdisk /dev/sda

    - command (m for help): p

    - **Delete partition 5 & 2**

        - command (m for help): d

        - partition munber (1,2,5, default 5): 5

            - partition 5 has been deleted

        - partition munber (1,2, default 2): 2

            - partition 2 has been deleted

        - command (m for help): p

            - You will observe only the root partition

        - command (m for help): w

            - The partition table has been altered.

            - Syncing disks.

#### 2.2 Expand/resize the root partition

- **Extend /dev/sda1:** To do this online while your server is running, we will use 'parted:'

- > sudo parted /dev/sda

    - (parted) resizepart

    - Partition number? 1

    - Are you sure you want to continue? Yes/No?  Yes

    - End? [n GB]? 200GB

        - n - is current partition size

        - 200GB - is total size of partition after extending.

    - (parted) quit

- **Verify the new disk layout:**

- > sudo fdisk -l

    - you will observer that the disk has been extended. 

- **Resize the filesystem**

- > sudo resize2fs /dev/sda1

    - provide your password

    - verify the resized filesystem

        - > df -lh

#### 2.3 Create the swap and extended partition

- Create second extended partition to carry the swap partition as logical volume:

- > sudo fdisk /dev/sda

    - command (m for help): n

    - Partition type (p & e)

    - select (default p): e

    - Partition number (2-4, default 2): 2

    - Hit 'Enter' for default values

    - command (m for help): w

        - Syncing disks.

- Create the logical volume for swap:

- > sudo fdisk /dev/sda

    - command (m for help): n

    - Hit 'Enter' for default values

    - **Change the partition type:**

        - command (m for help): t

        - Partition number (1,2,5, default 5): 5

        - Hex code : L

        - Hex code : 82

        - command (m for help): w

        - SYncing disks

        - **Verify**

            - > sudo fdisk -l

- **Enable the swap partition**

- > sudo mkswap /dev/sda5

    - If this command gives you an error (no such file or directory) -> 'REBOOT' the server.

    - execute the command again after rebooting

- > sudo swapon /dev/sda5

    - **Verify the swap**

        - sudo swapon --show

- **Make your swap space permanent**

    - Get the UUID for your newly created partition

    - > sudo blkid

        - Observe and 'COPY' the UUID of /dev/sda5

        - Open the fstab (sudo nano /etc/fstab) and 'PASTE' the UUID on the swap entry.

        - 'UNCOMMENT' the swap entry and 'SAVE' and 'EXIT' the fstab

        - **Verify the fstab configuration**

            - > sudo cat /etc/fstab

## REBOOT and VERIFY

- sudo swapon --show



