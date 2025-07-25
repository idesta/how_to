# Configuring Site-to-Site Strongswan IPSec between two Linux (Ubuntu) EC2 instances

This guide explains how to configure a Site-to-Site IPSec VPN between two EC2 instances using StrongSwan on Ubuntu.

- Launch two EC2 instances in different regions (use free tier eligible)

    - N-Virginia EC2

    - Europe-London EC2 

    - In order to access them via `SSH` you have to configure key-pair and download it to your local machine.


## Configuration Details

- N-Virginia EC2:
  - Public IP: `54.91.24.127`
  - Local Network: `172.31.32.0/20` VPC->Subnets->Select your Subnet which your private IP falls into.
  - Local IP: `172.31.45.241`

- Europe-London EC2:
  - Public IP: `18.130.84.132`
  - Subnet Network: `172.31.16.0/20` -> VPC->Subnets->Select your Subnet which your private IP falls into.
  - Private IP: `172.31.22.114`

- **Before you set up IPSec, make sure your two EC2 instances can communicate by checking if they can ping each other.**

  - 
    ```bash
    ping 172.31.45.241    from Europe-London EC2 Instance
    ```
  - 
    ```bash
    ping 172.31.22.114    from N.Virginia EC2 Instance
    ```
  - The expected output result:  Both instances won't be able to ping each other.

- Pre-Shared Key: Generate a secure PSK to be used by the peers using the following command using your local machine:

  ```bash
  openssl rand -base64 24
  ```
  ```plaintext
  output will look like: fzMo1sSGL2qgOPCmHzXFs4HsAMZ5Dp63
  ```
## Installation

Follow these steps on both Node8 and Node9:

1. `SSH` to your ubuntu EC2 instances

   ```bash
   ssh -i ipsec-demo.pem ubuntu@54.91.24.127
   ```

  ```bash
   ssh -i ipsec-demo-2.pem ubuntu@18.130.84.132

   ```
2. Update the system on both London and N.Virginia EC2 instances:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. Enable IPv4 packet forwarding:
   - Edit the configuration file:
     ```bash
     sudo nano /etc/sysctl.conf
     ```
   - Add the following lines at the end of configuration file on both EC2 instances:
     ```plaintext
     net.ipv4.ip_forward = 1
     net.ipv4.conf.all.accept_redirects = 0
     ```
   - Save the file and apply the changes (ctrl+o and ctrl+x) on both Instances:
     ```bash
     sudo sysctl -p
     ```

4. Install StrongSwan and dependencies on both instances:
   ```bash
   sudo apt install strongswan strongswan-pki libcharon-extra-plugins libcharon-extauth-plugins libstrongswan-extra-plugins libtss2-tcti-tabrmd0 -y
   ```
- Explanation: 
  - `strongswan`:
    The main package for the StrongSwan VPN software. It provides the core functionality for managing IPSec VPNs.

  - `strongswan-pki`:
    Provides tools to generate and manage Public Key Infrastructure (PKI) components such as certificates and keys. Useful when using certificate-based authentication.

  - `libcharon-extra-plugins`:
    Adds additional plugins to the Charon daemon, which is the core IKE (Internet Key Exchange) service for StrongSwan. These plugins expand the functionality, including advanced cryptographic algorithms and hardware offloading.

  - `libcharon-extauth-plugins`:
    Provides plugins for external authentication mechanisms, allowing integration with other authentication systems (e.g., RADIUS, EAP).

  - `libstrongswan-extra-plugins`:
    A set of additional plugins for StrongSwan that enhance its features, including handling non-standard protocols and cryptographic extensions.

  - `libtss2-tcti-tabrmd0`:
    A library for interfacing with Trusted Platform Modules (TPMs). It allows StrongSwan to use TPMs for secure storage of cryptographic keys.


5. Enable StrongSwan service to start on boot then start it on both instances:
   ```bash
   sudo systemctl enable strongswan-starter
   sudo systemctl start strongswan-starter
   ```

6. Verify service status on both instances:
   ```bash
   systemctl status strongswan-starter
   ```

## Minimal Configuration for Site-to-Site VPN

### Step 1: Configure N.Virginia EC2 Instance

1. Edit the IPSec configuration file:
   ```bash
   sudo nano /etc/ipsec.conf
   ```
   - Replace the file content with:

     ```plaintext

     config setup
      charondebug="ike 2, knl 2, cfg 2"
      uniqueids=yes
      strictcrlpolicy=yes


     conn site-to-site
      auto=add
      type=tunnel
      authby=secret
      left=%defaultroute
      leftid=54.91.24.127
      leftsubnet=172.31.45.241
      right=18.130.84.132
      rightsubnet=172.31.22.114/32
      ike=aes256-sha256-modp2048!
      esp=aes256-sha256-modp2048!
      keyexchange=ikev2
      keyingtries=0
      ikelifetime=24h
      lifetime=1h
      dpddelay=30
      dpdtimeout=120
      dpdaction=restart
      auto=start

     ```

2. Set up the pre-shared key:
   ```bash
   sudo nano /etc/ipsec.secrets
   ```
   - Add the following line:
     ```plaintext
     54.91.24.127 18.130.84.132 : PSK "fzMo1sSGL2qgOPCmHzXFs4HsAMZ5Dp63"
     ```

### Step 2: Configure Europe-London EC2

1. Edit the IPSec configuration file:
   ```bash
   sudo nano /etc/ipsec.conf
   ```
   - Replace the file content with:

     ```plaintext

     config setup
      charondebug="ike 2, knl 2, cfg 2"
      uniqueids=yes
      strictcrlpolicy=yes


     conn site-to-site
      auto=add
      type=tunnel
      authby=secret
      left=%defaultroute
      leftid=18.130.84.132
      leftsubnet=172.31.22.114
      right=54.91.24.127
      rightsubnet=172.31.45.241/32
      ike=aes256-sha256-modp2048!
      esp=aes256-sha256-modp2048!
      keyexchange=ikev2
      keyingtries=0
      ikelifetime=24h
      lifetime=1h
      dpddelay=30
      dpdtimeout=120
      dpdaction=restart
      auto=start

     ```

2. Set up the pre-shared key:
   ```bash
   sudo nano /etc/ipsec.secrets
   ```
   - Add the following line:
     ```plaintext
     18.130.84.132 54.91.24.127 : PSK "fzMo1sSGL2qgOPCmHzXFs4HsAMZ5Dp63"
     ```

### Step 3: Firewall and Security Group Configuration

- Ensure IPSec traffic is allowed through the firewall/Security Group on both EC2 Instances:

  - Security Group Configuration

    - On the N.Virginia EC2 Instance allow `ICMP`,`custom UDP` port 500, 4500 and `custom protocol` port 50 traffic that comes from Europe-London EC2 instance.

    - On the Europe-London EC2 Instance allow `ICMP`,`custom UDP` port 500, 4500 and `custom protocol` port 50 traffic that comes from N.Virginia EC2 instance.

    - Don't forget to allow `Custom TCP SSH` port 22 from Anywhere IPv4/Your Public IP.

- Ensure IPSec traffic is allowed through the inside firewall (ufw) on both EC2 Instances:

- Check whether the `UFW` is active or not.

    ```bash
    sudo ufw status
    ```
- If the output is just like below you can leave the below configuration.

  - However if you want also to secure from inside the server; you can activate and allow those rules.

  ```plaintext
  Status: inactive
  ```

  ```bash
  sudo ufw allow 500/udp
  sudo ufw allow 4500/udp
  sudo ufw allow proto esp
  ```

### Step 4: Restart StrongSwan Service

Restart the StrongSwan service on both EC2 instances to apply the changes:

```bash
sudo systemctl restart strongswan-starter
```

### Step 5: Verify IPSec Status

Check the status of the IPSec connections:

```bash
sudo ipsec statusall
```

```bash
sudo ipsec status
```
You should see output indicating that the connection is established with details about Security Associations (SAs).

### Step 6: Test Connectivity

Verify that the VPN is working by pinging a host in the remote subnet:

```bash
ping -c 4 <remote_host_in_subnet>
```

```bash
ping 172.31.45.241    from Europe-London EC2 Instance
```

```bash
ping 172.31.22.114    from N.Virginia EC2 Instance
```

If the ping succeeds, the VPN is configured correctly.

### Step 7: Check the Log for any error

```bash
journalctl -u strongswan-starter -f
```
---