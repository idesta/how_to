# how_to
Miscellaneous 'how to' procedures

# via ssh

> zergaw@israel-vostro3520:~$ cd ~/.ssh/

>zergaw@israel-vostro3520:~/.ssh$ ls
```
known_hosts  known_hosts.old
```
> ergaw@israel-vostro3520:~/.ssh$ ssh-keygen -o

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/zergaw/.ssh/id_ed25519): 
Enter passphrase for "/home/zergaw/.ssh/id_ed25519" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/zergaw/.ssh/id_ed25519
Your public key has been saved in /home/zergaw/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:maW8KnMYBKjIqOHQS/q0MMLTVpFZasVBRkuJd9p4aO0 zergaw@israel-vostro3520
The key's randomart image is:
+--[ED25519 256]--+
| .    OOo        |
|. .  ==+..       |
|=. . oo.B .      |
|*.o o. * O       |
|=+.o. . S        |
|*+oo.    E       |
|.=o. o  .        |
|  o + ..         |
|     +.          |
+----[SHA256]-----+

```
> zergaw@israel-vostro3520:~/.ssh$ ls
```
id_ed25519  id_ed25519.pub  known_hosts  known_hosts.old
```
> zergaw@israel-vostro3520:~/.ssh$ cat id_ed25519.pub
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJdH5TLuXjqBjf7d8o2q8AyXJka1bVnBa4kEh8ocaI1m 
```
>zergaw@israel-vostro3520

## Open VScode

> Go to the github repo directory

- Upgrade some parameter

- git add .

- git commit -m "ssh"

- git push

- check and confirm