# Docker GPU Setup Guide

Complete setup guide for Docker with NVIDIA GPU support on Ubuntu/Debian systems.

## Prerequisites

- Ubuntu/Debian-based Linux distribution
- NVIDIA GPU with proper drivers installed
- User account with sudo privileges (using `node` user in examples)
- Internet connection for package downloads

## Installation Steps

### 1. Install Docker

```bash
curl -fsSL https://get.docker.com/ -o get-docker.sh
sudo sh get-docker.sh
rm get-docker.sh
```

### 2. Add User to Docker Group

```bash
sudo usermod -aG docker node
```

*Replace `node` with your actual username if different.*

### 3. Install NVIDIA Container Toolkit

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt update
sudo apt install -y nvidia-container-toolkit
```

### 4. Configure Docker for GPU Support

```bash
sudo nvidia-ctk runtime configure --runtime=docker
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
docker run --rm --gpus all nvidia/cuda:12.0-base-ubuntu20.04 nvidia-smi
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
