# Updated Docker Installation Guide for Ubuntu 22.04 LXC

## Prerequisites
- Ubuntu 22.04 LXC container with **nesting enabled** in Proxmox
- Root or sudo access
- LXC container configured with: `features: nesting=1`

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

### 5. Install Docker & Docker Compose
```bash
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 6. Fix containerd.io Version Issue (Ubuntu 22.04)
```bash
# Downgrade to stable version that works properly in LXC
apt-get install -y --allow-downgrades containerd.io=1.7.28-1~ubuntu.22.04~jammy
apt-mark hold containerd.io
```

### 7. Configure Docker Network Security for LXC

**Critical Security Fix:** Docker's default iptables manipulation can expose containers beyond LXC boundaries, creating potential IPv4 forwarding vulnerabilities.

Create Docker daemon configuration:
```bash
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  "iptables": true,
  "ip-forward": true,
  "userland-proxy": false,
  "bridge": "docker0"
}
EOF
```

Apply sysctl network hardening:
```bash
cat >> /etc/sysctl.conf <<EOF
# Docker LXC Security
net.ipv4.ip_forward=1
net.ipv4.conf.all.forwarding=1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
EOF

sysctl -p
```

### 8. Start Docker Service
```bash
systemctl enable docker
systemctl start docker
systemctl restart containerd docker
```

### 9. Verify Installation
```bash
# Check Docker version
docker --version

# Check Docker Compose version
docker compose version

# Test Docker
docker run --rm hello-world

# Test Docker Compose (optional)
mkdir -p /tmp/compose-test && cd /tmp/compose-test
cat > compose.yaml <<EOF
services:
  test:
    image: hello-world
EOF

docker compose up
cd ~ && rm -rf /tmp/compose-test
```

## Quick Install Script

For a complete automated installation with security fixes:

```bash
#!/bin/bash
# Docker + Docker Compose Installation for Ubuntu 22.04 LXC

# Update system
apt update && apt upgrade -y

# Install prerequisites
apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker repository
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker and Docker Compose
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Fix containerd version for 22.04
apt-get install -y --allow-downgrades containerd.io=1.7.28-1~ubuntu.22.04~jammy
apt-mark hold containerd.io

# Configure Docker daemon
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  "iptables": true,
  "ip-forward": true,
  "userland-proxy": false,
  "bridge": "docker0"
}
EOF

# Apply network security settings
cat >> /etc/sysctl.conf <<EOF
net.ipv4.ip_forward=1
net.ipv4.conf.all.forwarding=1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
EOF
sysctl -p

# Start services
systemctl enable docker
systemctl start docker
systemctl restart containerd docker

# Verify
echo "=== Docker Version ==="
docker --version
echo "=== Docker Compose Version ==="
docker compose version
echo "=== Testing Docker ==="
docker run --rm hello-world
```

## Add User to Docker Group (Optional)
Only needed for non-root users:
```bash
usermod -aG docker $USER
newgrp docker
```
