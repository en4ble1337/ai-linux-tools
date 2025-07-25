# NVIDIA Driver Installation Guide for Ubuntu 22.04 LTS (CLI)

This guide provides simple instructions for installing NVIDIA drivers on a fresh Ubuntu 22.04 server or desktop system using the command-line interface (CLI). This method uses Ubuntu's standard repositories, which is the safest and most reliable approach.

-----

## Prerequisites

Before you start, make sure your system is ready.

1.  **Update Your System:** Open a terminal and run the following command to update your package list and upgrade any existing software.
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```
2.  **Check Your GPU:** Verify that your system has an NVIDIA GPU.
    ```bash
    lspci | grep -i nvidia
    ```
    You should see an output listing your NVIDIA graphics card. If you don't see any output, your system may not have an NVIDIA GPU.

-----

## Step 1: Check for Available Drivers

Ubuntu has a built-in tool that can detect your graphics card and show available drivers.

▶ Run the following command in your terminal:

```bash
ubuntu-drivers devices
```

You'll see a list of available drivers, with one usually marked as `recommended`. Note the names of the available driver packages for the next step.

**Example Output:**

```
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00002489sv00001462sd00003975bc03sc00i00
vendor   : NVIDIA Corporation
driver   : nvidia-driver-535 - distro non-free
driver   : nvidia-driver-575 - distro non-free
driver   : nvidia-driver-550 - distro non-free recommended
driver   : xserver-xorg-video-nouveau - distro free builtin
```

In this example, `nvidia-driver-550` is **recommended**, but other versions like `nvidia-driver-535` and `nvidia-driver-575` are also available.

-----

## Step 2: Install the NVIDIA Drivers

Based on the list from Step 1, choose one of the two options below.

### Option 1: Automatic Installation (Recommended) 👍

This is the easiest method. It automatically installs the driver version that Ubuntu has tested and marked as **recommended** for your hardware.

▶ Run this command to automatically install the best driver:

```bash
sudo ubuntu-drivers autoinstall
```

### Option 2: Manual Installation of a Specific Driver

If you need a specific version (e.g., for compatibility with software like CUDA), you can install it manually from the list generated in Step 1.

▶ For example, to install version `575`, you would use the following command. Just replace the package name with your desired driver.

```bash
sudo apt install nvidia-driver-575
```

> **Note on Secure Boot:** Regardless of the option you choose, if you have Secure Boot enabled, the installer will ask you to create a password (a "MOK password"). You'll need this password in the next step.

-----

## Step 3: Reboot the System

A reboot is **required** to load the new driver.

▶ Run the reboot command:

```bash
sudo reboot
```

  - **If you set a Secure Boot/MOK password:** After rebooting, a blue screen titled **Perform MOK Management** will appear.
    1.  Select **Enroll MOK**.
    2.  Select **Continue**.
    3.  When prompted, enter the password you created during the installation.
    4.  Select **Reboot**. Your system will restart again.

-----

## Step 4: Verify the Installation ✅

After your system reboots and you log back in, you can verify that the driver was installed correctly.

▶ Open a terminal and run the `nvidia-smi` (NVIDIA System Management Interface) command:

```bash
nvidia-smi
```

If the installation was successful, you will see a table with details about your NVIDIA driver version and GPU.

**Example Output:**

```
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 575.10.01              Driver Version: 575.10.01    CUDA Version: 12.6     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA GeForce RTX 4080        Off | 00000000:01:00.0 Off |                  N/A |
|  0%   40C    P8              15W / 320W |      1MiB / 16384MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
```
