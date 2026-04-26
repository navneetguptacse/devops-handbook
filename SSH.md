![Ubuntu](https://img.shields.io/badge/Ubuntu-Server-E95420)
![Linux](https://img.shields.io/badge/Linux-Server-FCC624)
![SSH](https://img.shields.io/badge/SSH-Key%20Authentication-blue)
![UTM](https://img.shields.io/badge/UTM-Virtual%20Machine-lightgrey)
![Mode](https://img.shields.io/badge/Mode-Headless%20%7C%20GUI-orange)
![Platform](https://img.shields.io/badge/Platform-macOS-lightgrey)
![Authentication](https://img.shields.io/badge/Authentication-Public%20Key-green)
![Level](https://img.shields.io/badge/Level-Beginner--Friendly-success)

# Ubuntu Linux Server VM SSH Setup Guide

This guide explains how to create an Ubuntu Server virtual machine, enable temporary root SSH access with a password, add an SSH public key, disable password login, and finally connect to the VM using an SSH private key.

## Requirements

- Ubuntu Server ISO image
- Virtual machine software such as UTM, VMware, VirtualBox, or any other VM manager
- At least 2 vCPU, 2 GB RAM, and 25 GB disk space
- Host machine terminal access
- VM network access

## Download Ubuntu Server ISO

Download the Ubuntu Server image from:

```text
https://cdimage.ubuntu.com/releases/
```

For Ubuntu 26.04 Server, open:

```text
https://cdimage.ubuntu.com/releases/26.04/release/
```

Select the server install image for your architecture.

Example image:

```text
ubuntu-26.04-live-server-arm64.iso
```

Use the ARM64 image for Apple Silicon or other ARM-based systems. Use the AMD64 image for most Intel or AMD systems.

## Create The Virtual Machine

Open your VM software and create a new virtual machine using the downloaded Ubuntu Server ISO.

Recommended minimum configuration:

| Resource | Minimum |
| -------- | ------- |
| CPU      | 2 vCPU  |
| RAM      | 2 GB    |
| Disk     | 25 GB   |

You can increase CPU, RAM, and disk size based on your workload.

During Ubuntu installation, configure:

- Username
- Password
- Hostname
- Disk layout
- Network settings

After installation completes, restart the VM and log in using the username and password created during installation.

## Update Ubuntu Packages

Run the following command inside the VM:

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

## Set Root User Password

Set a password for the root user:

```bash
sudo passwd root
```

Enter and confirm the new root password.

## Temporarily Enable Root SSH Login With Password

Open the SSH server configuration file:

```bash
sudo vi /etc/ssh/sshd_config
```

Find this line:

```text
#PermitRootLogin prohibit-password
```

Change it to:

```text
PermitRootLogin yes
```

Save the file and restart SSH:

```bash
sudo systemctl restart ssh
```

## Find The VM IP Address

Inside the VM, run:

```bash
ip addr
```

Find the IP address assigned to the VM network interface.

Example:

```text
192.168.64.10
```

## Login As Root Using Password

From your host machine, connect to the VM:

```bash
ssh root@<your-vm-ip>
```

Example:

```bash
ssh root@192.168.64.10
```

When prompted, type `yes`, then enter the root password.

## Create SSH Key Pair On Host Machine

On your host machine, create an SSH key pair:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

When asked for the file location, enter a custom path or press Enter to use the default location.

Example custom path:

```text
/Users/username/Downloads/ssh-key
```

After the command completes, two files are created:

| File          | Purpose     |
| ------------- | ----------- |
| `ssh-key`     | Private key |
| `ssh-key.pub` | Public key  |

Keep the private key safe. The private key is required to access the VM after password login is disabled.

To view the keys:

```bash
cat /Users/username/Downloads/ssh-key
cat /Users/username/Downloads/ssh-key.pub
```

## Add Public Key To The VM

Open a new terminal and log in to the VM as root using the root password:

```bash
ssh root@<your-vm-ip>
```

Create the root SSH directory:

```bash
mkdir -p /root/.ssh
```

Open the authorized keys file:

```bash
vi /root/.ssh/authorized_keys
```

Paste the full content of your public key file into this file.

The public key usually starts with:

```text
ssh-ed25519
```

Save the file.

Set the correct permissions:

```bash
chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys
```

## Enable Key-Based Login And Disable Password Login

Open the SSH server configuration file:

```bash
vi /etc/ssh/sshd_config
```

Make sure these values are set:

```text
PermitRootLogin prohibit-password
PasswordAuthentication no
PubkeyAuthentication yes
```

Save the file and restart SSH:

```bash
systemctl restart ssh
```

Exit the SSH session:

```bash
exit
```

## Login Using Private SSH Key

From your host machine, connect to the VM using the private key:

```bash
ssh -i <private-ssh-key-path> root@<your-vm-ip>
```

Example:

```bash
ssh -i /Users/username/Downloads/ssh-key root@192.168.64.10
```

If the login succeeds, the VM is now configured for SSH key-based root access and password-based SSH login is disabled.

## Troubleshooting

Check SSH service status inside the VM:

```bash
systemctl status ssh
```

Restart SSH after any configuration change:

```bash
systemctl restart ssh
```

Check the SSH configuration file for syntax errors:

```bash
sshd -t
```

If the private key has insecure permissions on the host machine, fix it with:

```bash
chmod 600 <private-ssh-key-path>
```
