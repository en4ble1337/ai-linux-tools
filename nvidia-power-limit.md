# NVIDIA GPU Power Limit Auto-Configuration

Automatically set the NVIDIA GPU power limit to 200 watts on Ubuntu 22.04 system startup.

## Overview

This guide shows how to configure your system to automatically apply a 200-watt power limit to your NVIDIA GPU every time Ubuntu 22.04 reboots using a cron job.

## Quick Setup

Run the following command to add the cron job without overwriting existing crontab entries:

```bash
(sudo crontab -l 2>/dev/null | grep -v "nvidia-smi"; echo "@reboot sleep 30 && /usr/bin/nvidia-smi -pl 200") | sudo crontab -
```

## Command Breakdown

- `sudo crontab -l 2>/dev/null`: Lists current root cron jobs, suppressing errors if no crontab exists
- `;`: Command separator for running multiple commands on one line
- `echo "@reboot /usr/bin/nvidia-smi -pl 200"`: Generates the new cron job entry
  - `@reboot`: Special cron string that runs the command once at startup
  - `/usr/bin/nvidia-smi -pl 200`: Sets GPU power limit to 200 watts using full path (recommended for cron jobs)
- `|`: Pipe operator that passes output from left command to right command
- `sudo crontab -`: Installs the combined crontab entries, reading from standard input (`-`)

## Verification

Verify the cron job was added successfully:

```bash
sudo crontab -l
```

You should see the new entry: `@reboot /usr/bin/nvidia-smi -pl 200`

## Persistence approach

# NVIDIA GPU Power Limit Persistence Guide

A simple guide to set persistent power limits on NVIDIA GPUs using nvidia-persistenced.

## Overview

This method uses NVIDIA's persistence daemon to maintain GPU power limit settings across reboots, eliminating the need for cron jobs or manual reapplication after system restarts.

## Prerequisites

- NVIDIA GPU with driver installed
- Root/sudo access
- `nvidia-smi` command available

## Installation Steps

### Step 1: Enable NVIDIA Persistence Daemon

```bash

# Check your NVIDIA driver version first
nvidia-smi | head -3

# Install the persistence daemon (replace XXX with your driver version if needed)
sudo apt update
sudo apt install nvidia-persistenced

# Or try the generic package
sudo apt install nvidia-utils-575

sudo systemctl enable nvidia-persistenced
sudo systemctl start nvidia-persistenced

### Step 2: Set Power Limit with Persistence

```bash
sudo nvidia-smi -pl 200 -pm ENABLED
```

Replace `200` with your desired power limit in watts.

### Step 3: Clean Up Existing Cron Jobs (Optional)

If you previously used cron jobs to set power limits:

```bash
sudo crontab -r
```

**Warning**: This removes ALL cron jobs for the root user. If you have other important cron jobs, use:

```bash
sudo crontab -e
```

And manually remove only the nvidia-smi lines.

## Verification

Check that the power limit is applied:

```bash
nvidia-smi
```

Look for the power usage line showing your set limit (e.g., `36W / 200W`).

Verify persistence daemon is running:

```bash
sudo systemctl status nvidia-persistenced
```

## Troubleshooting

### Power limit resets after reboot
- Check if nvidia-persistenced is enabled: `sudo systemctl is-enabled nvidia-persistenced`
- Verify the service started successfully: `sudo systemctl status nvidia-persistenced`

### Permission denied errors
- Ensure you're using `sudo` for all commands
- Check that your user has sudo privileges

### Service not found
- Install nvidia-utils package: `sudo apt install nvidia-utils-xxx` (replace xxx with your driver version)
- Or install the full NVIDIA driver package

## Alternative Power Limits

Common power limit values:
- Conservative: 150W-200W
- Moderate: 250W-300W  
- High Performance: 350W+

Check your GPU's maximum power limit:
```bash
nvidia-smi -q -d POWER
```

## Why This Method Works

Unlike temporary solutions (cron jobs, manual setting), nvidia-persistenced:
- Automatically applies settings when the driver loads
- Survives system reboots and driver reloads  
- Is the officially recommended approach by NVIDIA
- Requires no additional scripting or scheduling

## License

This guide is provided as-is for educational purposes.
