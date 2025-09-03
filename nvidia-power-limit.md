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

# NVIDIA RTX 5090 Power Limit Persistence Fix

This solution creates a systemd service to persistently set the NVIDIA RTX 5090 power limit to 500W on boot using persistence mode.

## Solution: Enable Persistence Mode + Systemd Service

### Step 1: Create the power limit script

```bash
sudo nano /usr/local/sbin/nv-power-limit.sh
```

Add this content:

```bash
#!/usr/bin/env bash
# Set power limits on NVIDIA GPU with persistence mode

# Make sure nvidia-smi exists
command -v nvidia-smi &> /dev/null || { echo >&2 "nvidia-smi not found ... exiting."; exit 1; }

POWER_LIMIT=500

# Enable persistence mode first
/usr/bin/nvidia-smi --persistence-mode=1

# Wait a moment for persistence mode to activate
sleep 2

# Set the power limit
/usr/bin/nvidia-smi --power-limit=${POWER_LIMIT}

exit 0
```

### Step 2: Make the script executable

```bash
sudo chmod 755 /usr/local/sbin/nv-power-limit.sh
```

### Step 3: Create the systemd service

```bash
sudo nano /etc/systemd/system/nvidia-power-limit.service
```

Add this content:

```ini
[Unit]
Description=Set NVIDIA GPU Power Limit with Persistence
After=multi-user.target
Wants=nvidia-persistenced.service

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/nv-power-limit.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

### Step 4: Enable and start the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable nvidia-power-limit.service
sudo systemctl start nvidia-power-limit.service
```

## Verification Commands

Check service status:
```bash
sudo systemctl status nvidia-power-limit.service
```

Verify persistence mode:
```bash
nvidia-smi -q | grep "Persistence Mode"
```

Check power limit:
```bash
nvidia-smi
```

## How It Works

- **Persistence Mode**: Keeps the NVIDIA driver loaded and maintains GPU settings
- **Systemd Dependencies**: Ensures proper service ordering and dependency management  
- **OneShot Service**: Runs once on boot and remains active to maintain the configuration
- **Power Limit**: Sets the RTX 5090 to 500W (down from default 575W)

This approach is more reliable than cron jobs because it properly handles driver dependencies and ensures the NVIDIA driver is fully initialized before applying power settings.

[1](https://github.com/systemd/systemd)
[2](https://github.com/giobauermeister/systemd-service-template)
[3](https://github.com/roadrunner-server/docs/blob/master/app-server/systemd.md)
[4](https://www.reddit.com/r/bearapp/comments/aidr83/syntax_highlight_for_systemd_files/)
[5](https://github.com/janisadhi/Dummy_Systemd_Service)
[6](https://gist.github.com/beingadityak/ab2e46988cccc0a5e6dd289065551d8d)
[7](https://github.com/kaxap/systemd)
[8](https://github.com/systemd/systemd/blob/master/docs/HOME_DIRECTORY.md)
[9](https://discourse.nixos.org/t/nix-maid-systemd-native-dotfile-management/64619)
