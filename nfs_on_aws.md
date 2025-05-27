# NFS on AWS EC2 (ubuntu 24.04)

### 1. Create and Launch 3 EC2 instances
- 1 EC2 Instance for nfs-server
- 2 EC2 Instances for nfs-client
- Note that on the nfs-server EC2 instance Security group nfs-port (2049)should be whitelisted inside the security group inbound-rule.
    - add custom port 2049 to be accesses from 0.0.0.0/0

### 2. Connect to the nfs-server dedicated EC2 Instance and install NFS

- ssh -i [keypair] ubuntu@[publicip]

- **sudo apt update && apt upgrade -y** [update the os]

- **sudo apt install -y nfs-kernel-server** [install the nfs-server package]

- **sudo mkdir -p /mnt/app-data** [create the directory to be accessed from other sources]

- **sudo chown nobody:nogroup /mnt/app-data** (Anyone has permission to access this directory)

- **sudo nano /etc/exports** (go to export config file)

- Insert this **/mnt/app-data 172.31.0.0/16(rw,sync,no_subtree_check)** to the above export config file.

    - NFS works with private IP

    - The NFS Server and NFS Clients must be in the same VPC or VPC Peering has to be configured.

- **sudo exportfs -a** -> Export the shared directory (/mnt/app-data)

- - **sudo exportfs -v** -> Check the exported shared directory

- **sudo systemctl enable nfs-server** (Enable nfs server)

- **sudo systemctl start nfs-server** (Start nfs server)

-  **sudo systemctl status nfs-server** (check the status nfs server)


### 3. Connect to one of the nfs client dedicated EC2 Instance and install NFS client and access the shared directory.

- ssh -i [keypair] ubuntu@[publicip]

- **sudo apt update && apt upgrade -y** [update the os]

- **sudo apt install nfs-common** [install the nfs package]

- sudo mkdir -p /mnt/shared-data-1

- sudo mount 172.31.16.179:/mnt/app-data /mnt/shared-data-1
    - change the private IP to the nfs-server private IP

- Make the Mount Persistent

    - **sudo nano /etc/fstab** and insert the below at the end

    - 172.31.16.179:/mnt/app-data /mnt/shared-data-1 nfs defaults 0 0
        - change the private ip to the nfs-server private IP


### 4. Follow the same procedure for the second nfs-client EC2 Instance

### 5. Create file on the nfs-server ec2 instance and place it on the exported shared path (/mnt/app-data)

- Go to /mnt/app-data directory

- echo "Welcome to nfs" > nfs.txt

- cat nfs.txt

### 6. Check whether you found the [nfs.txt] file inside the nfs-client instances (both EC2) after accessing them and go to their shared path (/mnt/shared-data-1) and (/mnt/shared-data-2)

- Go to (/mnt/shared-data-1) on the first nfs-client instance

- list the files exixted on that directory and you will get the file that was created at nfs-server EC2 instance (nfs.txt)

- Do the same on the second nfs-client instance.


