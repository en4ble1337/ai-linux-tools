## GNOME: Monitor Stays Black After Standby at 120/144 Hz (USB-C → DisplayPort Workaround)

A workaround for Intel laptops on GNOME/Wayland where a high-refresh ultrawide, connected by **USB-C → DisplayPort**, **stays black after monitor standby** at 120/144 Hz but wakes fine at 60 Hz.

The fix: a small user service that switches the monitor to **60 Hz when the screen locks** and back to **144 Hz after unlock**.

> ⚠️ **Experimental.** This works around a kernel driver bug. It does not fix it.

---

### 1. Symptoms

* Monitor on a direct USB-C to DisplayPort cable, set to 120 Hz or 144 Hz.
* GNOME locks or blanks the screen and the monitor goes into standby.
* On wake the monitor **stays black**. Typing the password blind does not help.
* Only **unplugging/replugging the cable** (or power-cycling the monitor) brings the picture back.
* At **60 Hz** the monitor wakes normally.

Journal during the failure:

```
gnome-shell: Page flip failed: drmModeAtomicCommit: Invalid argument
gnome-shell: Failed to post KMS update: drmModeAtomicCommit: Invalid argument
gnome-shell: Failed to use stored monitor configuration: Invalid mode 5120x1440 (143.984) for monitor 'SAM Odyssey G91F'
```

Check your own logs:

```bash
journalctl -b --no-pager | grep -E "drmModeAtomicCommit|Invalid mode"
```

---

### 2. Why 60 Hz Helps

| Mode | Pixel clock | Bandwidth at 8 bit/colour | Fits DP 1.4 HBR3 ×4 (≈25.9 Gbit/s)? |
|---|---|---|---|
| 5120×1440 @ 144 Hz | 1132.75 MHz | ≈27.2 Gbit/s | No, needs DSC |
| 5120×1440 @ 120 Hz | 944.00 MHz | ≈22.7 Gbit/s | Yes, 87 % full |
| 5120×1440 @ 60 Hz | 472.00 MHz | ≈11.3 Gbit/s | Yes, easily |

**Likely cause (not confirmed):** during standby the DisplayPort link drops and the driver lowers its link limits. On wake the high-refresh mode no longer fits, so every screen update is rejected. 60 Hz still fits. After unlock the monitor is fully awake, and a fresh mode change to 144 Hz trains a new full-speed link.

---

### 3. Requirements

* GNOME on Wayland with **`gdctl`** (ships with Mutter, GNOME 48+)
* `gdbus` (GLib, installed by default)
* systemd user session
* Optional: `notify-send` (`libnotify-bin`) for the failure notification

```bash
gdctl show
```

---

### 4. Find Your Monitor Name and Modes

```bash
gdctl show --modes
```

Note the **`Product:`** line and the exact mode strings (e.g. `5120x1440@59.996`, `5120x1440@143.984`). Edit `PRODUCT`, `MODE_LOCKED` and `MODE_UNLOCKED` in the script below if your monitor is different.

---

### 5. Install the Script

```bash
mkdir -p ~/.local/bin && cat > ~/.local/bin/g9-refresh-switch <<'EOF'
#!/usr/bin/env bash
# Workaround for the Odyssey G9 not waking at high refresh after monitor standby.
# Screen locks   -> switch the G9 to 60 Hz (wakes reliably).
# Screen unlocks -> switch the G9 back to 144 Hz.
# Only acts when the G9 is the only active display, so it never turns off the
# laptop screen or another monitor. Changes are temporary (not saved).

PRODUCT="Odyssey G91F"
MODE_LOCKED="5120x1440@59.996"
MODE_UNLOCKED="5120x1440@143.984"

log() { echo "$*"; }

# Prints the connector name (e.g. DP-3) of the G9 if it is the only active display.
g9_connector() {
    gdctl show 2>/dev/null | awk -v p="$PRODUCT" '
        /^Logical monitors:/          { inlog = 1 }
        !inlog && /Monitor [A-Za-z]+-[0-9A-Za-z-]+ /{ match($0, /Monitor [^ ]+/); cur = substr($0, RSTART + 8, RLENGTH - 8) }
        !inlog && index($0, "Product: " p) { g9 = cur }
        inlog && /Logical monitor #/     { nlog++ }
        inlog && /Monitors: \(/          { if ($0 !~ /\(1\)/) multi = 1 }
        inlog && g9 != "" && index($0, g9 " (") { g9active = 1 }
        END { if (g9 != "" && g9active && nlog == 1 && !multi) print g9 }'
}

set_mode() {
    local mode=$1 conn
    for try in 1 2 3; do
        conn=$(g9_connector)
        if [ -z "$conn" ]; then
            log "G9 not the only active display; skipping $mode"
            return 0
        fi
        if gdctl set -L -p -M "$conn" -m "$mode" 2>&1; then
            log "Set $conn to $mode"
            return 0
        fi
        log "Try $try: could not set $conn to $mode"
        sleep 3
    done
    notify-send -i video-display "G9 refresh switch" "Could not set $mode. Use Settings > Displays." 2>/dev/null
    return 1
}

gdbus monitor --session --dest org.gnome.ScreenSaver --object-path /org/gnome/ScreenSaver |
while read -r line; do
    case "$line" in
        *ActiveChanged\ \(true,\)*)
            set_mode "$MODE_LOCKED" ;;
        *ActiveChanged\ \(false,\)*)
            sleep 2   # let the monitor finish waking up
            set_mode "$MODE_UNLOCKED" ;;
    esac
done
EOF
chmod +x ~/.local/bin/g9-refresh-switch
```

**How it works:**
* **Lock (`org.gnome.ScreenSaver.ActiveChanged = true`):** sets the monitor to 60 Hz before it goes dark.
* **Unlock (`ActiveChanged = false`):** waits 2 s, then sets 144 Hz.
* **If 144 Hz is rejected:** retries 3 times, then shows a notification and stays at 60 Hz (no black screen).
* **Only acts when this monitor is the only active display.** `gdctl set` replaces the whole layout, so with the laptop screen or a second monitor on, it does nothing.
* **Finds the monitor by product name**, so any USB-C port works.
* **Temporary changes only.** `~/.config/monitors.xml` is not touched.

---

### 6. Start It Automatically at Login

```bash
mkdir -p ~/.config/systemd/user && cat > ~/.config/systemd/user/g9-refresh-switch.service <<'EOF'
[Unit]
Description=Switch Odyssey G9 to 60 Hz while locked, 144 Hz after unlock
PartOf=graphical-session.target
After=graphical-session.target

[Service]
ExecStart=%h/.local/bin/g9-refresh-switch
Restart=always
RestartSec=5

[Install]
WantedBy=graphical-session.target
EOF
```

```bash
systemctl --user daemon-reload
```

```bash
systemctl --user enable --now g9-refresh-switch.service
```

---

### 7. Test It

1. Lock with **Super+L** and let the monitor sleep.
2. Wake it and unlock.
3. Check the log:

```bash
journalctl --user -u g9-refresh-switch -n 20 --no-pager
```

Expected:

```
Set DP-3 to 5120x1440@59.996
Set DP-3 to 5120x1440@143.984
```

---

### 8. Uninstall

```bash
systemctl --user disable --now g9-refresh-switch.service
```

```bash
rm ~/.local/bin/g9-refresh-switch ~/.config/systemd/user/g9-refresh-switch.service
```

---

### ⚠️ Important Notes

* GNOME turns the monitor off soon after **Super+L**. The script switches to 60 Hz at lock time, before that.
* If the screen is already black: power-cycle the monitor with its button, or replug the cable.
* If 144 Hz fails after unlock you stay at 60 Hz. Change it in **Settings → Displays**.
* Does not cover laptop suspend (lid close).
* Does nothing with more than one active display.

**Tested on:** Lenovo ThinkPad L14 Gen 6 (Core Ultra 5 225U, Arrow Lake-U, `i915`), Ubuntu 26.04.1, kernel 7.0.0-38, GNOME/Mutter 50.1 Wayland, Samsung Odyssey G9 G91F (5120×1440 @ 144 Hz), Cable Matters USB-C → DP 1.4 cable (no dock).
