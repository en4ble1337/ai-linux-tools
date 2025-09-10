# Docker GPU Setup Guide

Complete setup guide for Docker with NVIDIA GPU support on Ubuntu/Debian systems.

## Prerequisites

- Ubuntu/Debian-based Linux distribution
- NVIDIA GPU with proper drivers installed
- User account with sudo privileges (using `node` user in examples)
- Internet connection for package downloads

## Installation Steps

# Docker Installation Guide

## Method 1: Quick Install Script

### 1. Install Docker (Quick Method)
```bash
curl -fsSL https://get.docker.com/ -o get-docker.sh
sudo sh get-docker.sh
rm get-docker.sh
```

## Method 2: Manual Installation

### 1. Update Package Index
```bash
sudo apt update
```

### 2. Install Prerequisites
```bash
sudo apt install -y ca-certificates curl gnupg lsb-release
```

### 3. Create Keyrings Directory
```bash
sudo mkdir -p /etc/apt/keyrings
```

### 4. Add Docker's Official GPG Key
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

### 5. Add Docker Repository
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 6. Update Package Index Again
```bash
sudo apt update
```

### 7. Install Docker Engine
```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

### 8. Enable and Start Docker Service
```bash
sudo systemctl enable docker && sudo systemctl start docker
```

### 9. Verify Installation
```bash
sudo docker --version
```

### 10. Add User to Docker Group (Optional)
```bash
sudo usermod -aG docker $USER
```

> **Note:** After adding yourself to the docker group, log out and log back in for the changes to take effect.


### 2. Add User to Docker Group

```bash
sudo usermod -aG docker node
```

*Replace `node` with your actual username if different.*

### 3. Install NVIDIA Container Toolkit

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) && \
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg && \
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list && \
sudo apt update
```

### 4. Configure Docker for GPU Support

```bash
sudo apt install -y nvidia-container-toolkit && \
sudo nvidia-ctk runtime configure --runtime=docker && \
sudo systemctl restart docker
```

### 5. Apply Docker Group Changes

```bash
exit
su - node
```

*Or alternatively, you can reboot the system to ensure all group changes take effect.*

### 6. Test Docker GPU Access

```bash
sudo docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```

## Verification Commands

### Check Docker Installation

```bash
docker --version
docker run hello-world
```

### Check NVIDIA Container Toolkit

```bash
nvidia-ctk --version
```

### Check GPU Visibility in Docker

```bash
docker run --rm --gpus all nvidia/cuda:12.0-base-ubuntu20.04 nvidia-smi
```

Expected output should show your GPU information similar to running `nvidia-smi` on the host system.

## Troubleshooting

### Docker Permission Denied

If you get permission denied errors:

```bash
# Verify user is in docker group
groups $USER

# If not in docker group, add user and restart session
sudo usermod -aG docker $USER
newgrp docker
```

### GPU Not Accessible in Container

```bash
# Check NVIDIA drivers on host
nvidia-smi

# Verify container toolkit installation
nvidia-ctk --version

# Check Docker daemon configuration
sudo cat /etc/docker/daemon.json
```

### Container Runtime Issues

```bash
# Restart Docker service
sudo systemctl restart docker

# Check Docker service status
sudo systemctl status docker
```

## Additional Notes

- The NVIDIA Container Toolkit requires NVIDIA drivers to be installed on the host system
- Different CUDA versions are available; replace `12.0-base-ubuntu20.04` with your preferred version
- For production environments, consider using specific CUDA image tags rather than `latest`
- Some applications may require additional CUDA libraries or specific base images

## Common Docker GPU Commands

```bash
# Run container with all GPUs
docker run --rm --gpus all <image>

# Run container with specific GPU
docker run --rm --gpus device=0 <image>

# Run container with GPU limits
docker run --rm --gpus '"device=0,1"' <image>
```

## Security Considerations

- Running Docker containers with GPU access requires elevated privileges
- Ensure your Docker images are from trusted sources
- Consider using Docker's security features like user namespaces in production environments
