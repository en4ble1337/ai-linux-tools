# NVIDIA GPU Power Limit Auto-Configuration

Automatically set the NVIDIA GPU power limit to 200 watts on Ubuntu 22.04 system startup.

## Overview

This guide shows how to configure your system to automatically apply a 200-watt power limit to your NVIDIA GPU every time Ubuntu 22.04 reboots using a cron job.

## Quick Setup

Run the following command to add the cron job without overwriting existing crontab entries:

```bash
(sudo crontab -l 2>/dev/null; echo "@reboot /usr/bin/nvidia-smi -pl 200") | sudo crontab -
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

## Requirements

- Ubuntu 22.04
- NVIDIA GPU with installed drivers
- `nvidia-smi` utility available
- Root/sudo access

## Notes

- The power limit will be applied automatically on every system reboot
- Using the full path `/usr/bin/nvidia-smi` ensures the command works in cron's limited environment
- This method preserves any existing root crontab entries
