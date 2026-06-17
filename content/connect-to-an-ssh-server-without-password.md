---
title: "Connect to an SSH Server Without Password"
type: post
date: 2023-10-10T18:20:14+02:00
draft: false
---

## Overview

This guide explains how to set up public‑key authentication so you can log in to a remote server without entering a password each time.

### Why use key‑based authentication?
* **Security** – Keys are harder to brute‑force than passwords.
* **Convenience** – No need to type a password for every SSH session.
* **Automation** – Essential for scripts, CI/CD pipelines, and remote management.

## Prerequisites
* A Unix‑like client (Linux/macOS) with `ssh` and `ssh-keygen` installed.
* Access to the remote server with a user account and password.

## Step 1 – Generate an SSH key pair
```bash
ssh-keygen
```
* Press **Enter** to accept the default file location (`~/.ssh/id_rsa`).
* Leave the passphrase empty (press **Enter** twice) for password‑less usage.

You should see output similar to:

```text
Generating public/private rsa key pair.
Enter file in which to save the key (/home/me/.ssh/id_rsa): 
Created directory '/home/me/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/me/.ssh/id_rsa
Your public key has been saved in /home/me/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:ZOW6KfiX4kGQ/Fu0Lb2hUoRZO/wEtYTa5DmlKvG2y9I me@mycomputer
The key's randomart image is:
+---[RSA 3072]----+
|        oo+      |
|   . . =o*..     |
|    + o=O++      |
|    .o.=*O       |
|     oo.S.=      |
|    .o++ = o     |
|    .+=.+..      |
|    .oE+o        |
|     o=+         |
+----[SHA256]-----+
```

## Step 2 – Verify SSH access with password
```bash
ssh remoteuser@remoteserver
```
You should be prompted for the remote user’s password. Once logged in, exit with `exit`.

## Step 3 – Copy the public key to the server
```bash
scp ~/.ssh/id_rsa.pub remoteuser@remoteserver:.
```
After the transfer, log back in:

```bash
ssh remoteuser@remoteserver
```
Now you should see the prompt without a password.

## Step 4 – Append the key to `authorized_keys`
On the server, run:

```bash
cat id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
If `authorized_keys` did not exist, create it first:

```bash
touch ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys
```

## Step 5 – Test password‑less login
```bash
ssh remoteuser@remoteserver
```
You should now log in without being asked for a password.

## Troubleshooting
* **Permission errors** – Ensure `~/.ssh` and `authorized_keys` are owned by you and have permissions `700` and `600` respectively.
* **Wrong key** – Verify the public key content matches the one on the server.
* **Host key verification** – If you see a warning about the host key changing, remove the old entry from `~/.ssh/known_hosts`.

---
**References**
* [OpenSSH Manual – ssh-keygen](https://man.openbsd.org/ssh-keygen)
* [OpenSSH Manual – ssh](https://man.openbsd.org/ssh)
