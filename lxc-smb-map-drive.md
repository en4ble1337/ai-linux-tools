# SMB File Sharing Between Ubuntu 22.04 LXC and Windows 11 (Using Root User)

Below is the revised, end-to-end guide in GitHub markdown format, reflecting all key steps and lessons learned—especially the **critical registration and password setup of the root user in the Samba password database**. This addresses the exact scenario you faced and should prevent any ambiguity for future reference.

***

## SMB File Sharing: Ubuntu 22.04 LXC to Windows 11 (Root User Auth)

**Status:** ✅ Tested and Working  
**Last Updated:** November 2025

***

### Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Proxmox/LXC Container Setup](#proxmoxlxc-container-setup)
- [Samba Installation and Configuration](#samba-installation-and-configuration)
- [Registering the Root User in Samba (Critical)](#registering-the-root-user-in-samba-critical)
- [File Permissions](#file-permissions)
- [Windows 11 Configuration](#windows-11-configuration)
- [Mapping the Drive on Windows 11](#mapping-the-drive-on-windows-11)
- [Troubleshooting](#troubleshooting)
- [Lessons Learned](#lessons-learned)
- [Security Notes](#security-notes)

***

## Overview

This guide details a reliable, reproducible method for sharing files via SMB from an Ubuntu 22.04 LXC container (Proxmox unprivileged) to Windows 11, using the container's root account for authentication—covering all commands and noting pitfalls around Samba authentication and directory permissions.

***

## Prerequisites

- Ubuntu 22.04 unprivileged LXC container (“unprivileged: 1” set)
- Windows 11 system on the same network
- Samba installed in the LXC container
- SSH access to your container (or console access via Proxmox)
- The LXC container has a reachable IP (can ping from Windows)

***

## Proxmox/LXC Container Setup

Your container config (from `/etc/pve/lxc/<ID>.conf`) should look something like:

```ini
arch: amd64
unprivileged: 1
cores: 4
features: nesting=1
memory: 3024
net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=<mac>,ip=dhcp,type=veth
...
```

***

## Samba Installation and Configuration

### 1. Update and Install Samba

```bash
sudo apt update
sudo apt install samba samba-common-bin cifs-utils -y
```

### 2. Edit `/etc/samba/smb.conf`

Add or amend the `[root]` share:

```ini
[root]
    path = /root
    comment = Root Directory Share
    available = yes
    valid users = root
    read only = no
    browsable = no
    create mask = 0777
    directory mask = 0777
    force create mode = 0777
    force directory mode = 0777
    guest ok = no
```

In the `[global]` section, check/add the following:

```ini
[global]
    workgroup = WORKGROUP
    security = user
    map to guest = never
    min protocol = SMB2
    max protocol = SMB3
```

Save and exit the file.

***

## Registering the Root User in Samba (Critical)

> **You must add “root” to the Samba password database, and set a password! This is the most common missing step leading to failed logins.**

```bash
sudo smbpasswd -a root
```
Enter your chosen root SMB password (separate from system password).

Enable root in the Samba user DB:

```bash
sudo smbpasswd -e root
```

Verify root is listed:

```bash
sudo pdbedit -L
```
Should output `root:0:`

***

## File Permissions

Set generous directory permissions for the share (for testing/home use):

```bash
sudo chmod 777 /root
```

You may restrict this further (e.g., `chmod 755 /root`) once everything is working.

***

## Restart and Test Samba

```bash
sudo systemctl restart smbd nmbd
sudo systemctl status smbd nmbd
```

Test access locally:

```bash
sudo smbclient -L localhost -U root
```
Enter your root SMB password and ensure `[root]` appears in the share list.

***

## Windows 11 Configuration

1. **Open PowerShell as Administrator:**

```powershell
Set-SmbClientConfiguration -EnableInsecureGuestLogons $false -Force
Set-SmbClientConfiguration -RequireSecuritySignature $false -Force
```

> These steps ensure only authenticated connections are attempted, compatible with most Samba servers.

***

## Mapping the Drive on Windows 11

**Disconnect** any previous mappings first:

```cmd
net use Z: /delete /yes
```

**Map the drive** (File Explorer or CMD):

```cmd
net use Z: \\<LXC-IP>\root /user:root <your-smb-password> /persistent:yes
```
Or use “This PC > Map network drive” in File Explorer, and provide root/SMB password when prompted.

***

## Troubleshooting

- **Can’t connect, but can see share:**  
  Double-check you set the **Samba password** for `root` (`sudo smbpasswd -a root`)!
- **Can read, but not write:**  
  Set directory permissions to `777` (`sudo chmod 777 /root`) and confirm config.
- **Got “Your organization's security policies block unauthenticated guest access” error:**  
  Make sure you’re mapping with a username AND `EnableInsecureGuestLogons` is disabled in PowerShell.

Check logs with:
```bash
sudo tail -f /var/log/samba/log.smbd
```

***

## Lessons Learned

- **Registering root in the Samba user database (with `sudo smbpasswd -a root`) is critical.**  
  Without this, even perfectly-configured Samba shares will refuse authentication.
- File permissions TRUMP Samba config:  
  If Linux denies writes to the target directory, Samba cannot override.
- Windows often caches credentials and share states—disconnect shares and reconnect after changes for reliable testing.
- Using root works seamlessly for basic mapping and is practical for home/lab, but has obvious security implications (see below).

***

## Security Notes

- **Using root for file sharing is inherently risky—never expose such shares to larger networks, the Internet, or use in production environments!**
- For better security, create a dedicated non-root user and share a non-system directory.
- Always use strong, unique passwords.

***

**References validated November 2025.** This guide covers all observed stumbling blocks in real setups. Copy, fork, or PR on GitHub as needed!
