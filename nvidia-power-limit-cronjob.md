# NVIDIA GPU Auto Power Limiter

This is a simple Bash script designed to run as a cronjob on a Linux system (like Proxmox, Ubuntu, etc.). It automatically checks the power limit of your NVIDIA GPU every minute and resets it to a specific value if it has been changed.

This is useful for maintaining a consistent power draw for GPU-intensive tasks like machine learning, rendering, or mining, especially if other processes or reboots sometimes reset the power limit.

## Prerequisites

  * A Linux system with NVIDIA drivers installed.
  * The `nvidia-smi` command-line tool must be installed and functional.
  * Root privileges are required to set the power limit (`nvidia-smi -pl`) and to edit the system-wide or root crontab.

> **Note:** On systems like Proxmox, you are typically logged in as the `root` user by default, so you do not need to use the `sudo` command.

## Setup Instructions

Follow these steps to deploy the script and automate it.

### 1\. Create the Script

First, create the script file using a text editor like `nano`.

```bash
nano /root/gpu_power_check.sh
```

Copy and paste the following code into the file:

```bash
#!/bin/bash

# --- CONFIGURATION ---
# The target power limit in Watts you want to enforce.
TARGET_POWER_LIMIT=200

# --- SCRIPT LOGIC ---
# Do not edit below this line unless you know what you are doing.

# Get the current power limit using a query designed for scripting.
# This is much more reliable than parsing the full text output.
CURRENT_POWER_LIMIT=$(nvidia-smi --query-gpu=power.limit --format=csv,noheader,nounits)

# Exit if nvidia-smi failed to return a value.
if [ -z "$CURRENT_POWER_LIMIT" ]; then
  exit 1
fi

# Convert the value to a whole number (e.g., "200.00" becomes "200").
CURRENT_POWER_LIMIT=${CURRENT_POWER_LIMIT%.*}

# Check if the current limit is not equal to our target.
if [ "$CURRENT_POWER_LIMIT" -ne "$TARGET_POWER_LIMIT" ]; then
  # If it's different, set the power limit to our target.
  # Using the full path is a good practice for cronjobs.
  /usr/bin/nvidia-smi -pl "$TARGET_POWER_LIMIT"
fi
```

Save the file and exit the editor (press `CTRL + X`, then `Y`, then `Enter`).

### 2\. Make the Script Executable

The script needs execution permissions to run.

```bash
chmod +x /root/gpu_power_check.sh
```

### 3\. Test the Script (Optional but Recommended)

Run the script manually to ensure it works without errors.

```bash
/root/gpu_power_check.sh
```

If your power limit was not 200W, it should now be set, and you will see a confirmation message from `nvidia-smi`.

### 4\. Schedule the Cronjob

Finally, add the script to your crontab to run it automatically every minute.

1.  Open the crontab editor:

    ```bash
    crontab -e
    ```

    (If prompted, choose `nano` as your editor).

2.  Add the following line to the bottom of the file:

    ```
    * * * * * /root/gpu_power_check.sh >/dev/null 2>&1
    ```

3.  Save and exit the crontab editor (`CTRL + X`, `Y`, `Enter`).

The job is now active and will run every minute to enforce your power limit.

## How It Works

### The Script

  * `TARGET_POWER_LIMIT=200`: This variable at the top of the file is the only thing you should need to change. Set it to your desired power limit in Watts.
  * `nvidia-smi --query-gpu=...`: This command securely gets *only* the numerical value of the current power limit, avoiding text-parsing errors.
  * `${CURRENT_POWER_LIMIT%.*}`: This is a shell parameter expansion that removes the decimal part of the number (e.g., `200.00` becomes `200`) so it can be reliably compared as an integer.
  * `if [ "$CURRENT_POWER_LIMIT" -ne "$TARGET_POWER_LIMIT" ]`: This is the core logic. It checks if the current power limit is **n**ot **e**qual (`-ne`) to your target. If they are different, it runs the `nvidia-smi -pl` command to reset it.

### The Cronjob

  * `* * * * *`: This is the cron schedule for "run every minute of every hour of every day."
  * `/root/gpu_power_check.sh`: This is the command to be executed.
  * `>/dev/null 2>&1`: This is important for automation. It redirects all normal and error output from the script to `/dev/null` (a "black hole"), preventing cron from sending you an email every minute.
