# Docker Installation Script for Ubuntu 22.04 LXC

Simple Docker installation guide for Ubuntu 22.04 running in Proxmox LXC containers.

## Prerequisites

- Ubuntu 22.04 LXC container with nesting enabled
- Root or sudo access

## Installation Steps

### 1. Update System

```bash
apt update && apt upgrade -y
```

### 2. Install Prerequisites

```bash
apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

### 3. Add Docker GPG Key

```bash
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg
```

### 4. Add Docker Repository

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 5. Install Docker

```bash
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 6. Start Docker Service

```bash
systemctl enable docker
systemctl start docker
```

### 7. Verify Installation

```bash
docker --version
docker run hello-world
```

## One-Line Installation Script (2 ways)

```bash
curl -fsSL https://get.docker.com | sh && systemctl enable docker && systemctl start docker
```

```bash
wget -qO- https://get.docker.com |sh
```

## Add User to Docker Group (Optional) Only needed if running as non-root user:

```bash
usermod -aG docker $USER
newgrp docker
```
