# GPU Fan Temperature Management Guide

Automated GPU fan curve management to maintain optimal temperatures using the GPU_FAN_OC_Manager tool.

## Overview

This guide shows how to automatically manage GPU fan curves to maintain a target maximum temperature. The tool continuously monitors and adjusts fan speeds to keep your GPU at or below the specified temperature threshold.

## Quick Setup (One-Line Install)

This command will download, install, and configure the GPU fan manager to maintain a maximum temperature of 65°C:

```bash
bash -c "wget https://github.com/jjziets/GPU_FAN_OC_Manager/raw/main/set_fan_curve; chmod +x set_fan_curve; CURRENT_PATH=\$(pwd); nohup bash -c \"while true; do \$CURRENT_PATH/set_fan_curve 65; sleep 1; done\" > output.txt & (crontab -l; echo \"@reboot screen -dmS gpuManger bash -c 'while true; do \$CURRENT_PATH/set_fan_curve 65; sleep 1; done'\") | crontab -"
```

## Manual Setup

### 1. Download the Script

```bash
wget https://github.com/jjziets/GPU_FAN_OC_Manager/raw/main/set_fan_curve
chmod +x set_fan_curve
```

### 2. Test the Script

```bash
./set_fan_curve 65
```

Replace `65` with your desired maximum temperature in Celsius.

### 3. Run Continuously (Background)

```bash
CURRENT_PATH=$(pwd)
nohup bash -c "while true; do $CURRENT_PATH/set_fan_curve 65; sleep 1; done" > output.txt &
```

### 4. Add to Startup (Crontab)

```bash
(crontab -l; echo "@reboot screen -dmS gpuManger bash -c 'while true; do $CURRENT_PATH/set_fan_curve 65; sleep 1; done'") | crontab -
```

## Usage

### Basic Command

```bash
./set_fan_curve <max_temperature>
```

**Examples:**
```bash
./set_fan_curve 65    # Keep GPU at or below 65°C
./set_fan_curve 70    # Keep GPU at or below 70°C
./set_fan_curve 60    # Keep GPU at or below 60°C
```

## What the One-Line Command Does

1. **Downloads the script** from the GitHub repository
2. **Makes it executable** with proper permissions
3. **Starts monitoring immediately** in the background
4. **Creates output logs** in `output.txt`
5. **Adds automatic startup** via crontab using screen session

## Managing the Service

### Check if Running

```bash
ps aux | grep set_fan_curve
screen -ls  # Check for gpuManger session
```

### View Logs

```bash
tail -f output.txt
```

### Stop the Service

```bash
# Kill the background process
pkill -f set_fan_curve

# Or kill the screen session
screen -S gpuManger -X quit
```

### Restart the Service

```bash
CURRENT_PATH=$(pwd)
nohup bash -c "while true; do $CURRENT_PATH/set_fan_curve 65; sleep 1; done" > output.txt &
```

## Customization

### Change Target Temperature

Edit the crontab to use a different temperature:

```bash
crontab -e
```

Find the line with `@reboot screen -dmS gpuManger` and change the temperature value.

### Adjust Update Frequency

To change from 1-second updates to a different interval, modify the `sleep 1` value:

```bash
# For 5-second updates
while true; do $CURRENT_PATH/set_fan_curve 65; sleep 5; done
```

## Troubleshooting

### Script Not Found

Ensure you're in the correct directory:

```bash
ls -la set_fan_curve
pwd
```

### Permission Issues

Make sure the script is executable:

```bash
chmod +x set_fan_curve
```

### GPU Not Detected

Check if NVIDIA drivers and tools are installed:

```bash
nvidia-smi
nvidia-settings --version
```

### Crontab Issues

Verify the crontab entry was added:

```bash
crontab -l
```

## Requirements

- NVIDIA GPU with driver support
- `nvidia-smi` and `nvidia-settings` utilities
- Linux system with bash shell
- `screen` utility (for background management)
- `wget` for downloading the script

## Important Notes

- The script runs continuously and updates every second by default
- Temperature monitoring requires NVIDIA GPU drivers
- The tool automatically adjusts fan curves based on current temperature
- Background processes will continue after terminal closure
- Crontab ensures the service starts automatically on reboot

## Credits

This guide uses the GPU_FAN_OC_Manager tool developed by **jjziets**.

**Original Repository:** https://github.com/jjziets/GPU_FAN_OC_Manager

Special thanks to jjziets for creating and maintaining this useful GPU management tool.
