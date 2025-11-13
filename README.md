# AI Linux Tools
A collection of useful Linux tools and guides for AI development and NVIDIA GPU management.

## Table of Contents

### Network & Security
- [22.04 Netplan Issue](22.04-netplan-issue.md) - Resolves network configuration issues in Ubuntu 22.04 with netplan
- [Nmap CLI](nmap-cli.md) - Command-line network scanning and security auditing with nmap
- [Port Check Guide](port-check-guide.md) - Guide for checking and testing network port connectivity
- [Fail2ban](fail2ban.md) - Configure intrusion prevention to protect against brute-force attacks
- [Root SSH Enable](root-ssh-enable.md) - Enable root SSH access with security considerations
- [SSH Key Gen](ssh-key-gen.md) - Generate and manage SSH keys for secure authentication

### Docker & Containers
- [Docker LXC](docker-lxc.md) - Guide for running Docker containers within LXC environments
- [Docker NVIDIA](docker-nvidia.md) - Setup and configuration for NVIDIA GPU support in Docker containers
- [Docker User Privileges](docker-user-priv.md) - Configure Docker user permissions and non-root access

### NVIDIA GPU Management
- [NVIDIA Drivers](nvidia-drivers-non-50series.md) - Installation and management of NVIDIA graphics drivers on Linux (non-50 series)
- [Ubuntu 22.04 RTX 5090](ubuntu22.04-5090.md) - Specific installation guide for NVIDIA RTX 5090 on Ubuntu 22.04
- [NVIDIA Custom Fan Curve](nvidia-custom-fan-curve.md) - Configure custom GPU fan curves for optimal cooling performance
- [NVIDIA Power Limit](nvidia-power-limit.md) - Adjust GPU power consumption limits for performance or efficiency
- [NVIDIA Power Limit Cronjob](nvidia-power-limit-cronjob.md) - Automate power limit settings with cronjobs

### System Administration
- [Disable Unattended Upgrades](disable-unnattended.md) - Disables automatic system updates to prevent unexpected package changes
- [Partition Expand](partition-expand.md) - Resize and expand disk partitions without data loss
- [LXC SMB Map Drive](lxc-smb-map-drive.md) - Mount and access SMB/CIFS network shares in LXC containers
- [Proxmox VM QEMU Guest](proxmox-vm-qemu-guest.md) - Configure QEMU guest agent for Proxmox VMs
- [MOTD Enhancement](motd-enhacement.md) - Customize message of the day for better system information display

### Development & Best Practices
- [FAFO Versioning 101](fafo-versioning-101.md) - Best practices for version control and software deployment strategies
- [Vi Editor Noob 101](vi-editor-noob101.md) - Beginner's guide to using the Vi/Vim text editor
