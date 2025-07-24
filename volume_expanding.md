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

> **sudo swapon -s**