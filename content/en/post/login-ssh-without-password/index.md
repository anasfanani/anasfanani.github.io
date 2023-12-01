---
title: Login SSH Without Password
tags:
  - SSH
categories:
  - Linux
date: 2023-09-01 20:36:51+07:00
description: "Login SSH without password using public key authentication. "
---
SSH (Secure Shell) is a secure network protocol used to manage remote servers or network devices. SSH offers a higher level of security than other network protocols, such as Telnet or FTP.
One way to improve SSH security is to use public key authentication. Public key authentication uses a pair of keys: a public key and a private key. The public key is stored on the server, and the private key is stored on the local computer.
To login SSH without a password, you need to generate a pair of public and private keys using the `ssh-keygen` command. Then, you need to copy the public key to the server. After that, you can login SSH without entering a password.
Here are the steps to login SSH without a password:
1. Generate a pair of public and private keys on your local computer using the `ssh-keygen` command.
```
ssh-keygen
```
This command will create a pair of public and private keys in RSA format. You will be prompted to enter a file name for the public and private keys. You can use any name you want.
2. Copy the public key to the server. You can use the `ssh-copy-id` command to copy the public key to the server.
```
ssh-copy-id <user_name>@<server_address>
```
This command will prompt you to enter the password for the user account on the server.
3. Test public key authentication. You can test public key authentication by trying to login SSH to the server without entering a password.
```
ssh <user_name>@<server_address>
```
If public key authentication is successful, you will successfully login to the server without entering a password.
Here is an example of how to use the `ssh-copy-id` command:
```
ssh-copy-id -i ~/.ssh/id_rsa_my_computer user@example.com
```
This command will copy the public key from the file `~/.ssh/id_rsa_my_computer` to the user account `user` on the server `example.com`.
**Benefits of SSH login without password**
SSH login without password offers several benefits, including:
* More secure than SSH login with password.
* Easier and faster than SSH login with password.
* Can be used to login to the server automatically.
**Conclusion**
SSH login without password is a more secure and convenient method to access remote servers. By following the steps above, you can easily configure SSH login without password on your local computer.