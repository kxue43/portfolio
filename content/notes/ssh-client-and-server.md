---
author: Ke Xue
title: SSH Client and Server
date: 2024-11-05T21:31:53-05:00
draft: false
layout: docs
description: Set up SSH client and server.
tags: 
- Linux
- SSH
---

This page describes how to set up a Ubuntu machine as SSH server to allow clients to remote in.

## SSH Server

First install `openssh-server`.

```bash
sudo apt-get install openssh-server
```

Afterwards check if the `sshd` service is active.

```bash
sudo systemctl status sshd
```

## SSH Client

First generate an SSH key pair.

```bash
ssh-keygen -t ed25519 -C <label>
```

Add the *public key* to the SSH server, this time using password authentication of the user.

```bash
ssh-copy-id -i $PUBLIC_KEY_PATH ${USER}@${HOST}
```

Put the following section in `~/.ssh/config`. Replace `<***>` with the values of the same-named
environment variables above:

```
Host <HOST>
    User <USER>
    IdentityFile <PUBLIC_KEY_PATH>
```

Remote into the SSH server with the following command.

```bash
ssh ${USER}@${HOST}
```

## Optionally Disable SSH Password Authentication

After setting up SSH Key authentication for one user, we can optionally disable SSH password authentication
on the server altogether.

Open the file `/etc/ssh/sshd_config`.
Find the line with:

```
#PasswordAuthentication yes
```

Change it to:

```
PasswordAuthentication no
```

Then restart the `sshd` service.

```bash
sudo systemctl restart sshd
```

## Use SSH Agent Forwarding to Allow `git clone` on Server

When cloning a private GitHub repository over SSH *on the server*, the server needs access to the SSH private key.
It is not secure to place the SSH private key file directly on the server. The better option is
to enable SSH agent forwarding on the client and forward the private key from client to server when they are connected.

For example, to set up agent forwarding for `github.com` put the following contents in `~/.ssh/config`:

```
Host <server_ip_v4>
    ForwardAgent yes
Host github.com
    User git
    AddKeysToAgent yes
    IdentityFile <path_to_private_key_file>
```

The `ForwardAgent yes` option shares the client's private keys with the server whenever there is a connection,
so should be used with caution. In particular, it is best to use specific IP addresses or host names
in the first rule, instead of using wildcard.

## References

- [OpenSSH configuration](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring)
- [SSH Key authentication](https://code.visualstudio.com/docs/remote/troubleshooting#_quick-start-using-ssh-keys)
- [Troubleshooting SSH agent forwarding](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/using-ssh-agent-forwarding#troubleshooting-ssh-agent-forwarding)
