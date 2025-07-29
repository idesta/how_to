## How to attach volume to EC2 Instance (Ubuntu)

> **1. Create volume**

> **2. Attach to the EC2 instance**

* **Verify the attached volume**
    * SSH to/Connect to your EC2 instance
        > type **sudo su** (super priveledge)
        
        > type **lsblk** and you will see just like this

        ```
         
            xvda     202:0     0    8G  0 disk
            ├─xvda1  202:1     0    7G  0 part /
            ├─xvda14 202:14    0    4M  0 part
            ├─xvda15 202:15    0  106M  0 part /boot/efi
            └─xvda16 259:0     0  913M  0 part /boot
            xvdz     202:6400  0    4G  0 disk

        ```
* Create **Mount Point** after disk verification

    * Choose your specific location (for this demo we select the root path)

    * type **mkdir mnt**  ---> to create directory 

    * type **cd mnt**  ---> to be inside 'mnt' directory

    * type **mkdir data**  ---> to create 'data' directory inside 'mnt' folder


*  **Format the Volume and make a file system** on it

    * check whether it has a file system or not

        > type **sudo file -s dev/xvdz**

        ```
        if the output is [/dev/xvdz: data] there is no file system on it

        ```
    * Format with ext4 filesystem (there are numerous linux filesystems)

        > type **sudo mkfs -t ext4 /dev/xvdz**

        ```
        The output should look like 

        mke2fs 1.47.0 (5-Feb-2023)
        Creating filesystem with 1048576 4k blocks and 262144 inodes
        Filesystem UUID: 3c21f22a-db84-4fa4-bd83-dafa91256aff
        Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

        Allocating group tables: done
        Writing inode tables: done
        Creating journal (16384 blocks): done
        Writing superblocks and filesystem accounting information: done
        
        ```

    * Verify the file system of the disk by typing [sudo file -s /dev/xvdz]

        ```
            The output should look like

            /dev/xvdz: Linux rev 1.0 ext4 filesystem data, UUID=3c21f22a-db84-4fa4-bd83-dafa91256aff (extents) (64bit) (large files) (huge files)

        ```
* **Mount the disk to your mount point (/mnt/data)**

    > type **sudo mount /dev/xvdz /mnt/data**

    > Verify whether the disk is mounted or not.

    - type **df -h** the output should be like this

    ```
        Filesystem      Size  Used Avail Use% Mounted on
        /dev/root       6.8G  1.8G  5.0G  26% /
        tmpfs           479M     0  479M   0% /dev/shm
        tmpfs           192M  888K  191M   1% /run
        tmpfs           5.0M     0  5.0M   0% /run/lock
        /dev/xvda16     881M   79M  741M  10% /boot
        /dev/xvda15     105M  6.1M   99M   6% /boot/efi
        tmpfs            96M   12K   96M   1% /run/user/1000
        /dev/xvdz       3.9G   24K  3.7G   1% /mnt/data

    ```

    - You can observe that at the bottom of the output result; the disk (/dev/xvdz) is mounted on the (/mnt/data)

* **Configure Automatic Mount on Reboot / Persistent Storage**

    > Get the UUID of the volume

    - type **sudo blkid** and copy the UUID of your attached volume

    ```
    The output shoul look like 

    /dev/xvda16: LABEL="BOOT" UUID="2aeda221-cde7-4cff-aa4c-e3f564d1e144"
    BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="60f2ed2c-e176-41ad-a5a7-3d7841fcf30e"
    /dev/xvda1: LABEL="cloudimg-rootfs" UUID="abc14a03-b423-456c-b531-def4a6ed6dc5"
    BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="a8ef6bc6-0748-47d7-a4a8-b126c21ff84c"
    /dev/xvda15: LABEL_FATBOOT="UEFI" LABEL="UEFI" UUID="E27E-E2BB" BLOCK_SIZE="512" 
    TYPE="vfat" PARTUUID="2fe86da6-16e0-4ec4-beda-110ca0cb2cbd"
    /dev/loop1: BLOCK_SIZE="131072" TYPE="squashfs"
    /dev/xvda14: PARTUUID="50830c16-53e9-46ca-8adc-762abe409832"
    /dev/loop2: BLOCK_SIZE="131072" TYPE="squashfs"
    /dev/loop0: BLOCK_SIZE="131072" TYPE="squashfs"

    /dev/xvdz: UUID="3c21f22a-db84-4fa4-bd83-dafa91256aff" BLOCK_SIZE="4096" TYPE="ext4"

    ```
    
    - type **sudo nano /etc/fstab** and at the end of the fstab file insert this (you can use [tab] for spacing)

    ```
    LABEL=cloudimg-rootfs   /        ext4   discard,commit=30,errors=remount-ro     0 1
    LABEL=BOOT      /boot   ext4    defaults        0 2
    LABEL=UEFI      /boot/efi       vfat    umask=0077      0 1

    UUID=3c21f22a-db84-4fa4-bd83-dafa91256aff  /mnt/data   ext4    defaults        0 2

    ```

    - Save and exit (ctrl + o) then (ctrl + x)

    - check whether its saved or not

        - type **cat /etc/fstab** you can observe that its saved.

* **Check the data persistency**

    - create a simple text file and write simple text on it.

    - Go to the mount point directory (cd /mnt/data)

    - type **echo "This is a test to check data persistence after reboot." | sudo tee test_persistence.txt**

    - check the text file is existed or not. type **ls** and you will observe that

    - check the actual content insude the created text file. type **cat test_persistence.txt** and you will observe the actual text content.

    - you can create another text file. type **touch another.txt**

* **Reboot the EC2 Instance and check the text files are existed or not inside the mount point path (/mnt/data) after accessing the server again.**





