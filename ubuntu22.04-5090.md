# Complete NVIDIA RTX 5090 Driver Installation Guide for Ubuntu 22.04

> **Disclaimer**: This installation was successfully performed on Proxmox 9.0 with GPU passthrough, but the steps are identical for bare metal Ubuntu 22.04 installations.

## Prerequisites

- Ubuntu 22.04 LTS with HWE (Hardware Enablement) kernel
- RTX 5090 graphics card properly seated and connected
- Internet connection for package downloads
- Admin/sudo privileges

## Step-by-Step Installation

### Step 1: Fresh System Setup and Updates

Start with a basic Ubuntu 22.04 installation using the HWE kernel server option.

```bash
sudo apt update && sudo apt upgrade -y && sudo apt dist-upgrade -y && sudo apt install update-manager-core -y
```

**Reboot the system:**
```bash
sudo reboot
```

### Step 2: Install Development Tools and Graphics PPA

Install essential build tools and add the graphics drivers PPA for better hardware support:

```bash
sudo apt install build-essential -y
sudo add-apt-repository ppa:graphics-drivers/ppa -y
sudo apt update
```

### Step 3: Download and Install NVIDIA Driver

Download the official NVIDIA driver version 575.64.05 (required for RTX 5090 support):

```bash
wget https://us.download.nvidia.com/XFree86/Linux-x86_64/575.64.05/NVIDIA-Linux-x86_64-575.64.05.run
chmod +x NVIDIA-Linux-x86_64-575.64.05.run
sudo ./NVIDIA-Linux-x86_64-575.64.05.run --dkms
```

### Step 4: Resolve Compiler Version Issues (If Needed)

If you encounter a compiler version mismatch error during installation (`cc: error: unrecognized command-line option '-ftrivial-auto-var-init=zero'`), install GCC 12:

```bash
sudo apt update
sudo apt install gcc-12 g++-12 -y
export CC=/usr/bin/gcc-12
```

Then re-run the NVIDIA installer:
```bash
sudo ./NVIDIA-Linux-x86_64-575.64.05.run --dkms
```
### GUI Steps for NVIDIA Accelerated Graphics Driver for Linux-x86_64

- MIT/GPL
- Alternate method of installing the NVIDIA drivers was detected - Continue
- Building kernel modules - Be patient
- No to 32bit libraries
- Register kernel module sources with DKMS - YES
- Rebuild initramfs
- No to Nvidia X driver

### Step 5: Final Reboot and Verification

Reboot the system to load the new drivers:
```bash
sudo reboot
```

Verify the installation by running:
```bash
nvidia-smi
```

## Expected Output

After successful installation, `nvidia-smi` should display output similar to:

```
Tue Sep  2 02:06:45 2025       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 575.64.05              Driver Version: 575.64.05      CUDA Version: 12.9     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 5090        Off |   00000000:01:00.0 Off |                  N/A |
| 30%   32C    P0             44W /  575W |       0MiB /  32607MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

## Important Notes

- **RTX 5090 Requirements**: Driver version 575.64.05 or higher is mandatory for RTX 5090 support
- **DKMS Flag**: Always use `--dkms` to ensure the driver rebuilds automatically after kernel updates
- **Compiler Compatibility**: Ubuntu 22.04's kernel is built with GCC 12, so GCC 12 must be used for module compilation
- **Memory**: The RTX 5090 features 32,607 MiB (32GB) of VRAM as shown in the output

## Troubleshooting

If the installation fails:
1. Check `/var/log/nvidia-installer.log` for detailed error messages
2. Ensure secure boot is disabled in BIOS/UEFI
3. Verify the GPU is properly connected and powered
4. Make sure no other NVIDIA drivers are installed (`sudo apt purge nvidia*` if needed)

## Post-Installation

Your RTX 5090 is now ready for:
- CUDA workloads
- Machine learning frameworks (TensorFlow, PyTorch)
- 3D rendering and gaming
- AI development and training

The installation provides CUDA 12.9 compatibility, ensuring support for the latest CUDA-accelerated applications.
