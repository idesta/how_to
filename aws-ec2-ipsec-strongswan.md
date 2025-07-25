# Configuring Site-to-Site Strongswan IPSec between two Linux (Ubuntu) EC2 instances

This guide explains how to configure a Site-to-Site IPSec VPN between two EC2 instances using StrongSwan on Ubuntu.

- Launch two EC2 instances in different regions (use free tier eligible)

    - N-Virginia EC2

    - Europe-London EC2 

    - In order to access them via `SSH` you have to configure key-pair and download it to your local machine

## Configuration Details

- N-Virginia EC2:
  - Public IP: `54.92.150.243`
  - Local Network: `172.31.32.0/20` VPC->Subnets->Select your Subnet which your private IP falls into.
  - Local IP: `172.31.32.78`

- Europe-London EC2:
  - Public IP: `18.133.233.201`
  - Subnet Network: `172.31.16.0/20` -> VPC->Subnets->Select your Subnet which your private IP falls into.
  - Private IP: `172.31.29.234`

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
   ssh -i ipsec-demo.pem ubuntu@54.92.150.243
   ```

  ```bash
   ssh -i ipsec-demo-2.pem ubuntu@18.133.233.201

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
         charondebug="ike 1, cfg 2"
         uniqueids=yes

     conn site-to-site
         type=tunnel                   # Configures a tunnel mode VPN.
         auto=start                    # Automatically starts the connection
         authby=secret                 # Use pre-shared key (replace with `authby=pubkey` if using certificates).
         left=54.92.150.243            # Public IP of N.Virginia EC2
         leftsubnet=172.31.32.0/20     # Private subnet of N.Virginia EC2
         right=18.133.233.201          # Public IP of Europe-London EC2
         rightsubnet=172.31.16.0/20    # Private subnet of Europe-London EC2
         ike=aes256-sha256-modp2048    # IKE Phase 1 parameters
         esp=aes256-sha256             # ESP parameters
     ```

2. Set up the pre-shared key:
   ```bash
   sudo nano /etc/ipsec.secrets
   ```
   - Add the following line:
     ```plaintext
     54.92.150.243 18.133.233.201 : PSK "fzMo1sSGL2qgOPCmHzXFs4HsAMZ5Dp63"
     ```

### Step 2: Configure Europe-London EC2

1. Edit the IPSec configuration file:
   ```bash
   sudo nano /etc/ipsec.conf
   ```
   - Replace the file content with:
     ```plaintext
     config setup
         charondebug="ike 1, cfg 2"
         uniqueids=yes

     conn site-to-site
         type=tunnel                   # Configures a tunnel mode VPN.
         auto=start                    # Automatically starts the connection
         authby=secret                 # Use pre-shared key (replace with `authby=pubkey` if using certificates).
         left=18.133.233.201           # Public IP of Europe-London EC2
         leftsubnet=172.31.16.0/20     # Private subnet of Europe-London EC2
         right=54.92.150.243           # Public IP of N.Virginia EC2
         rightsubnet=172.31.32.0/20    # Private subnet of N.Virginia EC2
         ike=aes256-sha256-modp2048    # IKE Phase 1 parameters
         esp=aes256-sha256             # ESP parameters
     ```

2. Set up the pre-shared key:
   ```bash
   sudo nano /etc/ipsec.secrets
   ```
   - Add the following line:
     ```plaintext
     18.133.233.201 54.92.150.243 : PSK "fzMo1sSGL2qgOPCmHzXFs4HsAMZ5Dp63"
     ```

### Step 3: Firewall and Security Group Configuration

- Ensure IPSec traffic is allowed through the firewall/Security Group on both EC2 Instances:

  - Security Group Configuration

    - On the N.Virginia EC2 Instance allow `ICMP`,`custom port` port 500, 4500 and `custom protocol` port 50 traffic that comes from Europe-London EC2 instance.

    - On the Europe-London EC2 Instance allow `ICMP`,`custom port` port 500, 4500 and `custom protocol` port 50 traffic that comes from N.Virginia EC2 instance.

- Ensure IPSec traffic is allowed through the inside firewall (ufw) on both EC2 Instances:

- Check whether the `UFW` is active or not.

    ```bash
    sudo ufw status
    ```
- If the output is just like below you can leave the below configuration.

  - However if you want to secure from inside the server also; you can activate and allow those rules.

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
sudo su
systemctl daemon-reload
sudo systemctl restart strongswan-starter
```

### Step 5: Verify IPSec Status

Check the status of the IPSec connections:

```bash
sudo ipsec statusall
```

You should see output indicating that the connection is established with details about Security Associations (SAs).

### Step 6: Test Connectivity

Verify that the VPN is working by pinging a host in the remote subnet:

```bash
ping -c 4 <remote_host_in_subnet>
```

If the ping succeeds, the VPN is configured correctly.

### Step 7: Check the Log for any error

```bash
journalctl -u strongswan-starter -f
```
---

This minimal setup provides a quick and efficient way to establish a Site-to-Site IPSec VPN using StrongSwan.

## Detailed Configuration for Site-to-Site VPN
### Configure Node8
   - Replace the file cont18.133.233.201 54.92.150.243 : PSK "fzMo1sSGL2qgOPCmHzXFs4HsAMZ5Dp63"ent with:
     ```plaintext
      config setup
          charondebug="ike 2, knl 2, cfg 2, esp 2, dmn 2, 2"  # Detailed debug levels for IKE, kernel, configuration, ESP, and daemon.
          uniqueids=yes

      conn site-to-site
          type=tunnel                   # Configures a tunnel mode VPN.
          auto=start                    # Automatically starts the connection
          authby=secret                 # Use pre-shared key (replace with `authby=pubkey` if using certificates).
          compress=no                   # aDisable compression for VPN traffic.
          dpdaction=restart             # aRestart the connection if the peer is unreachable.
          dpddelay=30s                  # aDead Peer Detection (DPD) delay interval.
          dpdtimeout=120s               # aDPD timeout before considering peer dead.
          rekey=yes                     # aAllow rekeying of the connection.
          reauth=no                     # aDisable reauthentication during rekeying.
          left=10.123.13.148             # Public IP of Node8
          leftsubnet=10.2.2.0/24         # Local subnet of Node8
          right=10.123.13.141            # Public IP of Node9
          rightsubnet=10.2.1.0/24        # Remote subnet of Node9
          ike=aes256-sha256-modp2048     # IKE Phase 1 parameters
          esp=aes256-sha256              # ESP parameters
          ikelifetime=8h                 # aLifetime for IKE SA (Phase 1) negotiation.
          lifetime=1h                    # aLifetime for IPsec SA (Phase 2) negotiation.
          margintime=3m                  # aRekey margin before SA expiration.
          keyingtries=3                  # aNumber of retries for key exchange.
          loglevel=2                     # aSets log verbosity (higher values increase logging).
     ```
### Configure Node9
   - Replace the file content with:
     ```plaintext
      config setup
          charondebug="ike 2, knl 2, cfg 2, esp 2, dmn 2, 2"  # Detailed debug levels for IKE, kernel, configuration, ESP, and daemon.
          uniqueids=yes

      conn site-to-site
          type=tunnel
          auto=start
          authby=secret
          compress=no                   # aDisable compression for VPN traffic.
          dpdaction=restart             # aRestart the connection if the peer is unreachable.
          dpddelay=30s                  # aDead Peer Detection (DPD) delay interval.
          dpdtimeout=120s               # aDPD timeout before considering peer dead.
          rekey=yes                     # aAllow rekeying of the connection.
          reauth=no                     # aDisable reauthentication during rekeying.
          left=10.123.13.141              # Public IP of Node9
          leftsubnet=10.2.1.0/24         # Local subnet of Node9
          right=10.123.13.148            # Public IP of Node8
          rightsubnet=10.2.2.0/24        # Remote subnet of Node8
          ike=aes256-sha256-modp2048     # IKE Phase 1 parameters
          esp=aes256-sha256              # ESP parameters
          ikelifetime=8h                 # aLifetime for IKE SA (Phase 1) negotiation.
          lifetime=1h                    # aLifetime for IPsec SA (Phase 2) negotiation.
          margintime=3m                  # aRekey margin before SA expiration.
          keyingtries=3                  # aNumber of retries for key exchange.
          loglevel=2                     # aSets log verbosity (higher values increase logging).
     ```

# IPsec Cryptographic Parameters
## 1. MODP (Modular Exponentiation) Groups
MODP groups are used for Diffie-Hellman key exchange, defined in RFC 3526 and RFC 5114. Higher bit sizes offer stronger security but require more computational power.

### Standard MODP Groups (RFC 3526)
| Group | Name      | Bit Size  | Keyword   |
|-------|-----------|-----------|-----------|
| 1     | MODP 768  | 768-bit   | `modp768` |
| 2     | MODP 1024 | 1024-bit  | `modp1024`|
| 5     | MODP 1536 | 1536-bit  | `modp1536`|
| 14    | MODP 2048 | 2048-bit  | `modp2048`|
| 15    | MODP 3072 | 3072-bit  | `modp3072`|
| 16    | MODP 4096 | 4096-bit  | `modp4096`|
| 17    | MODP 6144 | 6144-bit  | `modp6144`|
| 18    | MODP 8192 | 8192-bit  | `modp8192`|

### RFC 5114 MODP Groups (Alternative Prime Groups)
| Group | Name                          | Bit Size  | Keyword     |
|-------|-------------------------------|-----------|-------------|
| 22    | MODP 1024 (Safe Prime)        | 1024-bit  | `modp1024s` |
| 23    | MODP 2048 (Safe Prime)        | 2048-bit  | `modp2048s` |
| 24    | MODP 2048 (Alternative Prime) | 2048-bit  | `modp2048ap`|

## 2. Elliptic Curve Groups (RFC 5903)
Elliptic curve groups (ECP) provide stronger security at lower computational cost compared to MODP groups.

| Group | Name                 | Bit Size  | Keyword  |
|-------|----------------------|-----------|----------|
| 19    | ECP 256 (NIST P-256) | 256-bit   | `ecp256` |
| 20    | ECP 384 (NIST P-384) | 384-bit   | `ecp384` |
| 21    | ECP 521 (NIST P-521) | 521-bit   | `ecp521` |

## 3. Diffie-Hellman (DH) Groups for IPsec
DH groups are used for secure key exchange in IKE (Internet Key Exchange) within IPsec.

| Group | Type  | Bit Size | Security Level  | Keyword     |
|-------|-------|----------|-----------------|-------------|
| 14    | MODP  | 2048-bit | Secure          | `modp2048`  |
| 15    | MODP  | 3072-bit | Stronger        | `modp3072`  |
| 16    | MODP  | 4096-bit | Very Strong     | `modp4096`  |
| 19    | ECP   | 256-bit  | Strong (Faster) | `ecp256`    |
| 20    | ECP   | 384-bit  | Very Secure     | `ecp384`    |
| 21    | ECP   | 521-bit  | Highly Secure   | `ecp521`    |
| 23    | MODP  | 2048-bit | Secure          | `modp2048s` |

Best Practices:
- Use ECP groups (e.g., `ecp384`) for modern security.
- Avoid `modp768` and `modp1024` as they are weak.

## 4. Encryption Algorithms
Defines how IPsec traffic is encrypted.

| Algorithm           | Bit Size | Mode | Keyword           |
|---------------------|----------|------|-------------------|
| AES                 | 128-bit  | CBC  | `aes128`          |
| AES                 | 192-bit  | CBC  | `aes192`          |
| AES                 | 256-bit  | CBC  | `aes256`          |
| AES-GCM             | 128-bit  | AEAD | `aes128gcm16`     |
| AES-GCM             | 256-bit  | AEAD | `aes256gcm16`     |
| ChaCha20-Poly1305   | 256-bit  | AEAD | `chacha20poly1305`|
| 3DES (Deprecated)   | 168-bit  | CBC  | `3des`            |

Best Practices:
- Use `aes256gcm16` or `chacha20poly1305` for modern security.
- Avoid `3des`, as it's outdated and insecure.

## 5. Integrity (Hash) Algorithms
These algorithms ensure data integrity and authentication.

| Algorithm  | Bit Size | Keyword        |
|------------|----------|----------------|
| SHA1       | 160-bit  | `sha1`         |
| SHA2-256   | 256-bit  | `sha256`       | 
| SHA2-384   | 384-bit  | `sha384`       |
| SHA2-512   | 512-bit  | `sha512`       |
| AES-GMAC   | 128-bit  | `aes128gmac`   |
| AES-GMAC   | 192-bit  | `aes192gmac`   |
| AES-GMAC   | 256-bit  | `aes256gmac`   |

Best Practices:
- Use `sha256` or higher (`sha384` is better for long-term security).
- Avoid `sha1`, as it's vulnerable to attacks.

## 6. Key Exchange Version
IKE (Internet Key Exchange) manages key exchange in IPsec.

|        Version       | Keyword       |
|----------------------|---------------|
| IKEv1 (Legacy)       | `ikev1`       |
| IKEv2 (Recommended)  | `ikev2`       |

Best Practices:
- Use `ikev2` for modern security.
- IKEv1 should only be used for legacy systems.

## 7. Authentication Methods
Defines how peers authenticate in IPsec.

| Method               | Keyword                              |
|----------------------|--------------------------------------|
| Pre-Shared Key (PSK) | `psk`                                |
| X.509 Certificates   | `x509`                               |
| EAP Authentication   | `eap-mschapv2`, `eap-tls`, `eap-gtc` |

Best Practices:
- Prefer `x509` (certificate-based authentication) over `psk`.
- Use `eap-mschapv2` or `eap-tls` for user authentication.

## 8. IPsec Modes
Defines how IPsec packets are encapsulated.

| Mode           | Keyword       |
|----------------|---------------|
| Tunnel Mode    | `tunnel`      |
| Transport Mode | `transport`   |

Best Practices:
- Use `tunnel` mode for VPNs.
- Use `transport` mode when encrypting specific traffic.

## 9. Example Configuration in `ipsec.conf`
Example configuration using strong ciphers:
```ini
ike=aes256-sha256-modp2048
esp=aes256-sha256-modp2048
```
- With `!` → Only `modp2048` is allowed. No fallback to lower groups (`modp1024`, etc.).
- Without `!` → If `modp2048` is unavailable, the system may fallback to weaker groups.


# Errors
- The error message:
```
received AUTHENTICATION_FAILED notify error
```

indicates that the authentication phase of the IKE negotiation is failing. In other words, the remote peer is rejecting the authentication credentials (in this case, the pre-shared key) provided by your side.


# To be tested

If detailed logging is enabled in your StrongSwan configuration but you can't see the logs, the issue might be related to log storage, file permissions, or log verbosity settings. Here’s a step-by-step guide to troubleshoot and ensure you can view the logs:

---

### 1. Verify Logging Configuration
Make sure your `/etc/strongswan.conf` file has proper logging settings. A typical configuration for detailed logging looks like this:

```plaintext
charon {
    filelog {
        /var/log/strongswan.log {
            time_format = %b %e %T
            append = yes
            default = 2
            flush_line = yes
            ike = 2
            knl = 2
            net = 2
            esp = 2
        }
    }
    syslog {
        daemon {
            default = 1
            ike = 2
            cfg = 2
            net = 2
        }
    }
}
```

- Log Levels:
  - `0` - No logging.
  - `1` - Low-level information.
  - `2` - Informational logging (default for debugging).
  - `3` - Debugging (very detailed).

---

### 2. Check Permissions on Log Files
Ensure that StrongSwan can write to its log files:
```bash
sudo chmod 640 /var/log/strongswan.log
sudo chown root:adm /var/log/strongswan.log
```

If the file doesn’t exist, create it:
```bash
sudo touch /var/log/strongswan.log
sudo chmod 640 /var/log/strongswan.log
sudo chown root:adm /var/log/strongswan.log
```

---

### 3. Restart StrongSwan to Apply Changes
Restart the StrongSwan service to apply the logging configuration:
```bash
sudo systemctl restart strongswan-starter
```

---

### 4. Verify Logging Output
To check the logs:
- For file-based logs:
  ```bash
  tail -f /var/log/strongswan.log
  ```
- For syslog (if enabled):
  ```bash
  sudo journalctl -u strongswan-starter
  ```

---

### 5. Increase Logging Temporarily
If you need even more detailed logs, you can increase verbosity temporarily by editing `/etc/ipsec.conf`:

```plaintext
config setup
    charondebug="ike 2, knl 2, cfg 2, net 2, esp 2"
```

Restart the service after making changes:
```bash
sudo systemctl restart strongswan-starter
```

---

### 6. Ensure StrongSwan Debugging Tools Are Installed
Install the `strongswan-plugin-load-tester` package for more advanced debugging and performance insights:
```bash
sudo apt install strongswan-plugin-load-tester
```

---

### 7. Troubleshooting Tips
If you still cannot see logs:
1. Check if StrongSwan is running:
   ```bash
   sudo systemctl status strongswan-starter
   ```
2. Look for errors in the system logs:
   ```bash
   sudo journalctl -xe
   ```
3. Verify log rotation: Ensure logs are not being rotated or deleted prematurely by logrotate.





config setup 

    charondebug="ike 1, knl 1, cfg 0"  # Enable debugging for IKE, kernel, and configuration

    uniqueids=no  # Allow multiple connections from the same client



conn ikev2-vpn

    auto=add  # Automatically add this connection on startup

    type=tunnel  # Create a tunnel

    keyexchange=ikev2  # Use IKEv2 key exchange

    left=%any  # Allow connections from any source

    leftid=@yourdomain.com  # Client identifier (your domain)

    leftsubnet=0.0.0.0/0  # Allow all client subnets

    right=%any  # Allow connections to any destination

    rightid=%any  # Peer identifier (can be any)

    rightauth=eap-mschapv2  # Use MSCHAPv2 for authentication

    eap_identity=%identity  # Use username as EAP identity

    rightsourceip=10.0.0.0/8  # Assign a specific subnet to the client on connection

    # Add additional encryption and authentication options as needed

    ike=aes256-sha256-modp2048,aes128-sha1-modp1024

    esp=aes256-sha256,aes128-sha1
