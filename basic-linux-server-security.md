# Basic Linux Server Security

### 1. Implement strong passwords

- 
```plaintext
Ensure that admin and all user accounts on your cloud server, especially those with SSH access, have strong and unique passwords. 
Avoid using easily guessable usernames and passwords.
```
### 2. Disable root login via SSH

- 
```plaintext
Logging in directly as the root user via SSH is risky because it provides attackers with half the information they need to compromise your system. 
Instead, use a regular user account and then switch to the root user using su or sudo after login.
```
- To `disable root login via SSH`

    - Go to ssh config file

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

    - Find and uncomment the PermitRootLogin yes comment.

        - Change `#PermitRootLogin prohibit-password` to `PermitRootLogin yes`.

### 3. Implement SSH key authentication and disable password authentication

- 
```plaintext
When you implement Key based authentication, the trust domain will be transferred from password to
device. SSH access will be restricted to devices whose SSH public Key is registered on your cloud
server. 
```
- To implement SSH key authentication and disable password authentication on your cloud server, follow these steps:

    - Generate SSH Key Pair (on your local machine):

        - 
        ```plaintext
        If you haven't already generated an SSH key pair, you can do so using the ssh-keygen command. Open a terminal on your local machine and execute the following command
        ```

        - **# IN YOUR LOCAL MACHINE**

        ```bash
        sh-keygen -t rsa -b 4096
        ```

        ```plaintext
        Follow the default parameters for the key pair generation. You can also add strong passphrase additional to the public key authentication.
        ```
        - You would observe the below output
        ```
        ubuntu@ip-172-31-45-241:~$ ssh-keygen -t rsa -b 4096
                                Generating public/private rsa key pair.
                                Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): 
                                Enter passphrase (empty for no passphrase): 
                                Enter same passphrase again: 
                                Your identification has been saved in /home/ubuntu/.ssh/id_rsa
                                Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub
                                The key fingerprint is:
                                SHA256:EotI0xJF/nWlZLfrD3k9fN/cOdEDQStZALUP40unfSM ubuntu@ip-172-31-45-241
                                The key's randomart image is:
                                +---[RSA 4096]----+
                                |  .oo    .=o=o   |
                                |   +     o ++o.  |
                                |  + o . . o*...  |
                                | . + o + .. =o   |
                                |  . . + S  o.o. .|
                                |       .  ..=..+.|
                                |           o+E.==|
                                |             +ooO|
                                |              ..*|
                                +----[SHA256]-----+
        ```
        - 
        ```plaintext
        This command will generate a new SSH key pair using RSA algorithm with a key length of 4096 bits. Make sure you do not share the private key. It should only stay in the local machine and never shared to anyone.
        ```

        - 
        ```plaintext
        Copy the Public Key to the cloud server: After generating the SSH key pair, you need to copy the public key to your cloud server. 
        ```
        - 
        ```bash
        ssh-copy-id ubuntu@54.91.24.127
        ```

        - 
        ```plaintext
        Verify SSH Key Authentication: Once the public key is copied to the server, try logging in using SSH:
        ```

        - 
        ```bash
        ssh -i key-pair.pem ubuntu@54.91.24.127
        ```

        - 
        ```plaintext
        You should now be able to log in with( your passphrase) or without entering a password, as your SSH key will be used for authentication.
        ```

        - **IN YOUR CLOUD SERVER**

        - **Disable Password Authentication**

            - 
            ```plaintext
            After verifying that SSH key authentication works, you can disable password authentication to enhance security. Edit the SSH configuration file.
            ```

            - 
            ```bash
            sudo nano /etc/ssh/sshd_config
            ```
            - 
            ```plaintext
            Find the line PasswordAuthentication and set its value to no. If the line is commented out, uncomment it and set the value to no:
            ```
            - Change `#PasswordAuthentication yes` to `PasswordAuthentication no`

        - **Enable Public key Authentication**

            - 
            ```plaintext
            Find the line PubkeyAuthentication and set its value to yes. If the line is commented out, uncomment it and set the value to yes.
            ```
            - change `#PubkeyAuthentication yes` to `PubkeyAuthentication yes`

            - Save the file and exit the text editor.

            - After making changes to the SSH configuration, restart the SSH service to apply the changes:

                - 
                ```bash
                sudo systemctl restart ssh.service
                ```
        - 
        ```plaintext
        Test SSH login again to ensure that password authentication is disabled and only SSH key authentication is allowed. Once these steps are completed, SSH key authentication will be enforced, and password authentication will be disabled on your Ubuntu Server. This significantly enhances the security of your SSH service.
        ```

### 4. Configure SSH to use a non-standard port

- 
```plaintext
By default, SSH operates on port 22; changing the default SSH port can help reduce the number of automated attacks targeting your cloud server. However, this should not be your only security measure as determined attackers can still find the SSH port.
```
- Open a terminal on your cloud server and edit the SSH configuration file.

```bash
sudo nano /etc/ssh/sshd_config
```

- Locate the Port Configuration:
```plaintext
Within the sshd_config file, look for the line that specifies the port SSH listens on. By default, it is usually set to Port 22.
```

```plaintext
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```
- You'll need to change this to your desired non-standard port, such as: `9759`

- NB: you can set any port number.

- change `#Port 22` to `Port 9759`

- After making the change, save the file and exit the text editor.

- To apply the changes, restart the SSH service by executing the following command:

    - 
    ```bash
    sudo systemctl restart ssh.service
    ```

- **Update Firewall Rules (if applicable):**

    - 
    ```plaintext
    If you have a firewall enabled on your cloud server, such as UFW (Uncomplicated Firewall), you'll need to allow traffic on the new SSH port. For example, if you're using UFW, you can allow traffic on port 9759 by running:
    ```

    - 
    ```bash
    sudo ufw enable
    sudo ufw allow 9759/tcp
    sudo ufw status
    ```

    - 
    ```plaintext
    After restarting the SSH service and updating firewall rules (if necessary), test SSH connectivity by attempting to connect to your cloud server from your local machine using the new port.
    ```
    - 
    ```bash
    ssh -i key-pair.pem ubuntu@54.91.24.127 -p 9759
    ```

    - 
    ```plaintext
    By following these steps, you'll have configured SSH to listen on a non- standard port (9759) on your cloud Server. However, remember that changing the SSH port is just one layer of security, and you should implement other security measures in conjunction with this change.
    ```

### 5. FAIL TO BAN(fail2ban) Configuration
    
- Here's how to install and configure fail2ban on your cloud server to protect against SSH brute-force attacks:

- **Update and Install Fail2ban**

- 
```bash
sudo apt update
sudo apt install fail2ban
sudo systemctl status fail2ban.service
sudo systemctl enable fail2ban.service
sudo systemctl start fail2ban.service
```
- This command should show that the fail2ban service is running (active).

- **Configure Fail2ban for SSH (Optional):**

    - 
    ```plaintext
    By default, fail2ban comes with a pre-configured jail for SSH named sshd. However, you might want to review and potentially adjust some settings:
    ```
    - 
    ```bash
    sudo nano /etc/fail2ban/jail.d/defaults-debian.conf
    ```

    - Enter the following entries manually

    ```plaintext
    [sshd]
    enabled = true
    maxretry = 3
    bantime = 300
    ```

    ```plaintext
    If the sshd.conf file doesn't exist and if you want the ssh configuration would be handled separately use the following command and create one.
    ```

    ```bash
    sudo touch /etc/fail2ban/jail.d/sshd.conf
    ```
    - `maxretry:` This defines the number of failed login attempts before blocking the IP (in this scenario: 3).
    - `bantime:` This sets the duration (in seconds) that an IP address is blocked after exceeding the maxretrylimit (in this scenario: 300 → 5 minutes). 
    - Edit the values based on your needs. 
    - `Save and close` the file (Ctrl+O, then Ctrl+X).
    
    - **Restart Fail2ban**

        - 
        ```bash
        sudo systemctl restart fail2ban.service
        ```
    -  **Testing Fail2ban**

        - Warning: Perform this test from a separate machine or with caution to avoid accidentally locking yourself out.

        - From another machine, attempt SSH login to your cloud server with an incorrect password exceeding the maxretry limit set in the sshd.conf file 
        (e.g., 3 times). If fail2ban is working correctly, you should be unable to connect after exceeding the allowed attempts.


        - You can check the fail2ban logs for details:

            - 
            ```bash
            sudo fail2ban-client status sshd
            sudo nano /var/log/auth.log
            ```
- 
```plaintext
Fail2ban also offers jails for other services like FTP, Apache, etc. You can find configuration files for these in the /etc/fail2ban/jail.d/ directory.
Consider enabling email notifications from fail2ban to alert you about blocked IP addresses. Refer to the fail2ban documentation for details on setting this up. By implementing these steps, you'll have fail2ban actively monitoring your SSH login attempts and blocking suspicious activity, helping to secure your cloud server.
```

 ### 6. IP TABLES

 - 
 ```plaintext Protecting your cloud server using IP tables is a great way to enhance security. IP tables is a user-space utility program that allows you to configure the firewall (netfilter) provided by the Linux kernel. Here's a basic guide to setting up IP tables to allow access only from your IP and to block/reject malicious Ips.
 ```

 - Before any package installation updating/upgrading your server is highly recommended.

 - 
 ```bash
 sudo apt install iptables-persistent
 ```

 -  Before configuring IP table setup in your cloud server you have to know your trusted IP address.(it might be your public ip)

 - To `allow` your trusted IP address, if you are using a `standard ssh port`:
    - 
    ```bash
    sudo iptables -A INPUT -p tcp --dport 22 -s your_ip -j ACCEPT
    ```

 - To `allow` your trusted IP address, if you are using a `non-standard ssh port`:
    - 
    ```bash
    sudo iptables -A INPUT -p tcp --dport 9759 -s your_ip -j ACCEPT
    ```

 - To `reject` a suspicious IP address, if you are using a `standard ssh por`t`:
    - 
    ```bash
    sudo iptables -A INPUT -p tcp --dport 22 -s suspicious_ip -j DROP
    ```

 - To `allow` a suspicious IP address, if you are using a `non-standard ssh port`:
    - 
    ```bash
    sudo iptables -A INPUT -p tcp --dport 9759 -s suspicious_ip -j DROP
    ```

 - Once you've added the rules, **save them to persist across reboots:**

    ```bash
    sudo iptables-save | sudo tee /etc/iptables/rules.v4
    ```

    ```plaintext
    This will save the rules to /etc/iptables/rules.v4, which iptables-persistent will load automatically on boot. Make sure that you have added your current IP address to the trusted list before applying them. Otherwise, you may lock yourself out of SSH access.
    ```
 - Apply the IP tables rules

    - 
    ```bash
    sudo iptables-save 
    ```
    - check the IP tables rule

    ```bash
    sudo iptables -L
    ```

```plaintext
Finally we recommend that to continuously monitor your server's logs for suspicious activity, such as repeated failed login attempts or unusual access patterns. Promptly investigate and take action against any suspicious activity. Keep your cloud server updated with the latest security patches and software updates. Enable automatic updates to ensure critical patches are applied promptly, minimizing vulnerabilities. By implementing these security measures, you can significantly enhance the protection of your cloud server and minimize the risk of unauthorized access or compromise. Remember to stay vigilant and proactive in monitoring and updating your server's security measures to adapt to evolving threats.
```

```plaintext
Keep your cloud server updated with the latest security patches and software updates. Enable automatic updates to ensure critical patches are applied promptly, minimizing vulnerabilities. By implementing these security measures, you can significantly enhance the protection of your cloud server and minimize the risk of unauthorized access or compromise. Remember to stay vigilant and proactive in monitoring and updating your server's security measures to adapt to evolving threats.
```

```plaintext
By implementing these security measures, you can significantly enhance the protection of your cloud server and minimize the risk of unauthorized access or compromise. Remember to stay vigilant and proactive in monitoring and updating your server's security measures to adapt to evolving threats.
```