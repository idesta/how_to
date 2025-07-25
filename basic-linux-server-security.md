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

        - Disable Password Authentication

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

        - Enable Public key Authentication

            - 