
# Customizing the Ubuntu 22.04 MOTD: A Comprehensive Guide

This guide provides a step-by-step process for replacing the default Ubuntu 22.04 "Message of the Day" (MOTD) with a clean, custom, and highly informative login screen. The final result includes a custom welcome banner and a detailed, colorized system information panel that dynamically displays vital statistics, including optional NVIDIA GPU vitals.

## Final Result

Your new login screen will look similar to this:

```
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-157-generic x86_64)

*****************************************************************
* *
* Welcome to the Staging Environment Server           *
* *
* All connections are monitored and logged. Unauthorized   *
* access is strictly prohibited.                  *
* *
*****************************************************************

System information as of Wed Oct  8 01:45:12 PM CDT 2025

  OS Release:     Ubuntu 22.04.5 LTS          Users logged in: 1
  IPv4 address:   10.1.20.30 (ens18)          Processes:       162
  CPU Cores:      8                           Uptime:          2 hours, 28 minutes
  System load:    0.15

  Memory usage:   52.1% (2041M/3914M)
  Swap usage:     0.0% (0M/2047M)
  GPU Vitals:     NVIDIA GeForce RTX 4080 | Temp: 45°C | Fan: 30% | Power: 25.5W

Filesystem Usage:
  Mount Point               Size       Used       Avail      Use%       Filesystem
  /                         98G        8.5G       85G        9%         /dev/sda2
  /boot/efi                 511M       5.3M       506M       2%         /dev/sda1
  /mnt/data                 1.8T       750G       1.1T       42%        /dev/sdb1

```

## Table of Contents

1.  [How it Works](https://www.google.com/search?q=%23how-it-works)
2.  [Prerequisites](https://www.google.com/search?q=%23prerequisites)
3.  [Installation Steps](https://www.google.com/search?q=%23installation-steps)
      * [Step 1: Disable Default MOTD Scripts](https://www.google.com/search?q=%23step-1-disable-default-motd-scripts)
      * [Step 2: Add a Custom Welcome Banner](https://www.google.com/search?q=%23step-2-add-a-custom-welcome-banner)
      * [Step 3: Add the Advanced System Info Script](https://www.google.com/search?q=%23step-3-add-the-advanced-system-info-script)
4.  [Customization](https://www.google.com/search?q=%23customization)
5.  [Reverting Changes](https://www.google.com/search?q=%23reverting-changes)
6.  [Troubleshooting](https://www.google.com/search?q=%23troubleshooting)

-----

## How it Works

Modern Ubuntu systems dynamically generate the MOTD at login by executing scripts in the `/etc/update-motd.d/` directory in numerical order. This guide works by:

1.  **Disabling** the default, cluttered scripts.
2.  **Adding** a new script with a low number (`05-`) for a custom banner.
3.  **Adding** another script with a high number (`99-`) for a detailed system summary.

## Prerequisites

  * An instance of **Ubuntu Server 22.04 LTS**.
  * **sudo** or root privileges.
  * (Optional) For GPU monitoring, an **NVIDIA GPU** with drivers installed, making the `nvidia-smi` command available.

-----

## Installation Steps

Log into your server and follow these three steps.

### Step 1: Disable Default MOTD Scripts

First, we'll prevent the default Ubuntu scripts from running. This removes ads, news, and redundant information. We use `chmod -x` to make them non-executable, which is safer than deleting them.

```bash
# Disable the news, ads, and ESM/Pro info
sudo chmod -x /etc/update-motd.d/50-motd-news

# Disable the "Documentation," "Management," and "Support" links
sudo chmod -x /etc/update-motd.d/10-help-text

# Disable the default system information (we will replace it)
sudo chmod -x /etc/update-motd.d/50-landscape-sysinfo
```

### Step 2: Add a Custom Welcome Banner

Next, create a script to display your own static welcome message. This script is named `05-custom-message` to ensure it runs first.

1.  Create the file with `nano` or your preferred editor:

    ```bash
    sudo nano /etc/update-motd.d/05-custom-message
    ```

2.  Paste the following code into the file. You can edit the text inside the `printf` statements to your liking.

    ```sh
    #!/bin/sh
    # This script adds a custom welcome banner to the MOTD

    printf "\n"
    printf "*****************************************************************\n"
    printf "* *\n"
    printf "* Welcome to the Staging Environment Server           *\n"
    printf "* *\n"
    printf "* All connections are monitored and logged. Unauthorized   *\n"
    printf "* access is strictly prohibited.                  *\n"
    printf "* *\n"
    printf "*****************************************************************\n"
    printf "\n"
    ```

3.  Save the file and make it executable:

    ```bash
    sudo chmod +x /etc/update-motd.d/05-custom-message
    ```

### Step 3: Add the Advanced System Info Script

This is the main script that gathers and displays all the dynamic system information. It is named `99-sysinfo` to ensure it runs last.

1.  Create the file:

    ```bash
    sudo nano /etc/update-motd.d/99-sysinfo
    ```

2.  Paste the entire script below into the file.

    ```bash
    #!/bin/bash
    #
    # 99-sysinfo
    #
    # This script provides an enhanced, colorized summary of system information,
    # including optional GPU vitals and detailed disk utilization.

    # --- Color Definitions ---
    GREEN=$(tput setaf 2)
    YELLOW=$(tput setaf 3)
    BLUE=$(tput setaf 4)
    CYAN=$(tput setaf 6)
    WHITE=$(tput setaf 7)
    NC=$(tput sgr0) # No Color

    # --- Data Gathering ---
    # Find the primary network interface and its IPv4 address
    INTERFACE=$(ip route | grep default | awk '{print $5}')
    IP_ADDRESS=$(ip -4 addr show $INTERFACE | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
    IP_INFO="$IP_ADDRESS ($INTERFACE)"

    # OS Information & CPU Cores
    OS_INFO=$(grep PRETTY_NAME /etc/os-release | cut -d'"' -f2)
    CPU_CORES=$(nproc --all)

    # Uptime & Load
    UPTIME=$(uptime -p | sed 's/^up //')
    LOAD=$(cat /proc/loadavg | awk '{print $1}')

    # Memory and Swap Usage
    MEM_USAGE=$(free -m | awk 'NR==2{printf "%.1f%% (%sM/%sM)", $3*100/$2, $3, $2}')
    SWAP_USAGE=$(free -m | awk 'NR==3{printf "%.1f%% (%sM/%sM)", $3*100/$2, $3, $2}')

    # Users & Processes
    USERS=$(users | wc -w)
    PROCESSES=$(ps -e --no-headers | wc -l)

    # --- Optional GPU Vitals (for NVIDIA) ---
    GPU_INFO=""
    if command -v nvidia-smi &> /dev/null; then
        # Query the first GPU (index 0).
        GPU_DATA=$(nvidia-smi --query-gpu=gpu_name,temperature.gpu,fan.speed,power.draw --format=csv,noheader,nounits -i 0)
        if [ -n "$GPU_DATA" ]; then
            GPU_NAME=$(echo $GPU_DATA | cut -d',' -f1 | sed 's/^\s*//')
            GPU_TEMP=$(echo $GPU_DATA | cut -d',' -f2 | sed 's/^\s*//')
            GPU_FAN=$(echo $GPU_DATA | cut -d',' -f3 | sed 's/^\s*//')
            GPU_POWER=$(echo $GPU_DATA | cut -d',' -f4 | sed 's/^\s*//')
            # Format the final string for display
            GPU_INFO=$(printf "%s | Temp: %s°C | Fan: %s%% | Power: %sW" "$GPU_NAME" "$GPU_TEMP" "$GPU_FAN" "$GPU_POWER")
        fi
    fi

    # --- Display ---
    echo
    printf "${CYAN}System information as of %s${NC}\n" "$(date)"
    echo

    printf "  %-15s %s%-25s%s %-15s %s%s%s\n" "OS Release:" "$WHITE" "$OS_INFO" "$NC" "Users logged in:" "$YELLOW" "$USERS" "$NC"
    printf "  %-15s %s%-25s%s %-15s %s%s%s\n" "IPv4 address:" "$GREEN" "$IP_INFO" "$NC" "Processes:" "$YELLOW" "$PROCESSES" "$NC"
    printf "  %-15s %s%-25s%s %-15s %s%s%s\n" "CPU Cores:" "$WHITE" "$CPU_CORES" "$NC" "Uptime:" "$WHITE" "$UPTIME" "$NC"
    printf "  %-15s %s%-25s%s %-15s %s%s%s\n" "System load:" "$YELLOW" "$LOAD" "$NC"
    echo

    printf "  %-15s %s%s%s\n" "Memory usage:" "$YELLOW" "$MEM_USAGE" "$NC"
    printf "  %-15s %s%s%s\n" "Swap usage:" "$YELLOW" "$SWAP_USAGE" "$NC"

    # Display GPU info only if the variable is not empty
    if [ -n "$GPU_INFO" ]; then
        printf "  %-15s %s%s%s\n" "GPU Vitals:" "$GREEN" "$GPU_INFO" "$NC"
    fi

    # --- Filesystem Usage ---
    echo
    printf "${CYAN}Filesystem Usage:${NC}\n"
    # Use awk to format the output of df for readability
    df -hT | grep -vE 'tmpfs|squashfs|devtmpfs' | awk '
    BEGIN {
        printf "  ${BLUE}%-25s %-10s %-10s %-10s %-10s %-s${NC}\n", "Mount Point", "Size", "Used", "Avail", "Use%", "Filesystem";
    }
    NR>1 {
        printf "  %-25s %-10s %-10s %-10s %-10s %-s\n", $7, $3, $4, $5, $6, $1;
    }'
    echo
    ```

3.  Save the file and make it executable:

    ```bash
    sudo chmod +x /etc/update-motd.d/99-sysinfo
    ```

That's it\! The next time you log in, you will be greeted with your new, customized MOTD.

-----

## Customization

  * **Colors:** You can change the colors in the `99-sysinfo` script by modifying the color definition variables at the top (e.g., `GREEN=$(tput setaf 2)`).
  * **Layout:** The layout is controlled by the `printf` statements in the "Display" section of `99-sysinfo`. The `%-15s` format specifier creates a left-aligned string padded to 15 characters. Adjust these numbers to change column widths.
  * **Content:** You can add or remove information by adding new variables in the "Data Gathering" section and then adding a new `printf` line to display them.

-----

## Reverting Changes

To restore the original Ubuntu MOTD, simply remove your custom scripts and re-enable the default ones.

```bash
# Remove your custom scripts
sudo rm /etc/update-motd.d/05-custom-message
sudo rm /etc/update-motd.d/99-sysinfo

# Re-enable the default scripts
sudo chmod +x /etc/update-motd.d/10-help-text
sudo chmod +x /etc/update-motd.d/50-motd-news
sudo chmod +x /etc/update-motd.d/50-landscape-sysinfo
```

-----

## Troubleshooting

  * **Script doesn't run or shows errors:**
      * Ensure the script is executable: `sudo chmod +x /etc/update-motd.d/your-script-name`.
      * Ensure the first line is exactly `#!/bin/bash` or `#!/bin/sh`.
  * **GPU info isn't showing:**
      * This feature is only for NVIDIA GPUs.
      * Ensure the NVIDIA drivers are correctly installed and the `nvidia-smi` command works when run manually from the terminal.
  * **Colors don't display correctly:**
      * Your SSH client may not support colors. Check your terminal emulator's settings (e.g., PuTTY, Windows Terminal, iTerm2).
