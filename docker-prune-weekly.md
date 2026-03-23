
## Docker Automated Cleanup Strategies

Both commands append new automated tasks (cron jobs) to the system's crontab without overwriting existing jobs. They are designed to prevent Docker from consuming all available disk space by automatically removing unused containers and images.

### 1. Basic Weekly Time-Based Cleanup

This command sets up a scheduled task to indiscriminately clean up Docker resources once a week, regardless of current disk usage.

```bash
(crontab -l; echo "0 3 * * 0 docker container prune -f >> /var/log/docker-prune.log 2>&1 && docker image prune -a -f >> /var/log/docker-prune.log 2>&1") | crontab -
```

**How it works:**
* **Schedule (`0 3 * * 0`):** Runs every Sunday at 3:00 AM.
* **Containers:** Runs `docker container prune -f` to forcefully remove all stopped containers.
* **Images:** Runs `docker image prune -a -f` to forcefully remove **all** unused images (not just dangling ones).
* **Logging:** Appends all standard output and errors (`2>&1`) to `/var/log/docker-prune.log` for troubleshooting.

---

### 2. Threshold-Based Cleanup (80% Disk Utilization)

This command is more intelligent. It runs frequently but only executes the cleanup if the Docker storage directory is getting dangerously full.

```bash
(crontab -l; echo '0 */6 * * * USAGE=$(df /var/lib/docker | awk '\''NR==2 {print $5}'\'' | tr -d '\''%'\''); if [ "$USAGE" -gt 80 ]; then docker container prune -f >> /var/log/docker-prune.log 2>&1 && docker image prune -a -f >> /var/log/docker-prune.log 2>&1; fi') | crontab -
```

**How it works:**
* **Schedule (`0 */6 * * *`):** Evaluates every 6 hours (e.g., midnight, 6:00 AM, 12:00 PM, 6:00 PM).
* **Disk Check:**
    * `df /var/lib/docker`: Checks the disk space of the default Docker data directory.
    * `awk ... | tr -d '%'`: Extracts the exact percentage of space used and strips the `%` sign so it can be evaluated as an integer (e.g., `85`).
* **Condition (`if [ "$USAGE" -gt 80 ]`):** If the extracted disk usage is strictly greater than 80%, it triggers the cleanup.
* **Cleanup & Logging:** Executes the exact same container and image prune commands (and logging) as the weekly script.

---

### ⚠️ Important Notes for Operators
* **Permissions:** Writing to `/var/log/docker-prune.log` usually requires root privileges. Ensure this crontab is applied to the `root` user (via `sudo crontab -e`), or change the log path to a user-owned directory (like `~/.docker-prune.log`).
* **Image Re-downloads:** Because both scripts use the `-a` flag on `image prune`, Docker will delete images that aren't tied to currently running containers. If you spin those containers down and the cron runs, you will have to re-download the image layers the next time you start them.
