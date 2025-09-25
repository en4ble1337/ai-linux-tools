# Fail2ban Quick Setup Guide

Complete guide to install and configure Fail2ban for SSH protection on Ubuntu/Debian systems.

## What is Fail2ban?

**Fail2ban** is an intrusion prevention tool that protects your server from brute-force attacks by:

- 🔍 **Monitoring log files** (like `/var/log/auth.log`) for suspicious activity
- 🚫 **Automatically banning IP addresses** that make too many failed login attempts
- ⏰ **Temporary blocks** - bans are time-limited (default 10 minutes)
- 🔥 **Firewall integration** - uses iptables to block malicious IPs
- 🛡️ **Multi-service protection** - can protect SSH, web servers, email servers, etc.

### Why Use Fail2ban with SSH Keys?

Even with SSH key authentication, fail2ban provides:
- Protection against compromised keys
- Reduced log spam from bot attacks
- Defense against other services
- Multiple layers of security (defense in depth)

## Quick Installation

### One-Command Setup

```bash
# Install, configure, and start fail2ban
curl -sSL https://raw.githubusercontent.com/your-repo/fail2ban-setup.sh | bash
```

### Manual Installation

```bash
# Update system and install fail2ban
apt update && apt install -y fail2ban

# Enable and start the service
systemctl enable fail2ban
systemctl start fail2ban

# Check installation
systemctl status fail2ban
```

## Basic Configuration

### Create Custom Configuration

```bash
# Create local configuration (overrides defaults)
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
# Ban IP for 10 minutes after 3 failed attempts within 10 minutes
bantime = 600
findtime = 600
maxretry = 3

# Email notifications (optional)
# destemail = admin@yourdomain.com
# sendername = Fail2ban
# mta = sendmail

[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 1800
EOF

# Restart fail2ban to apply configuration
systemctl restart fail2ban
```

### Configuration Explained

| Setting | Description | Default Value |
|---------|-------------|---------------|
| `bantime` | How long to ban an IP (seconds) | 600 (10 minutes) |
| `findtime` | Time window to count failures (seconds) | 600 (10 minutes) |
| `maxretry` | Failed attempts before ban | 3 |
| `port` | Port to monitor | 22 (SSH) |
| `logpath` | Log file to monitor | `/var/log/auth.log` |

**Example:** With default settings, if an IP fails to login 3 times within 10 minutes, it gets banned for 10 minutes.

## Management Commands

### Check Status

```bash
# Overall status
fail2ban-client status

# SSH jail status  
fail2ban-client status sshd
```

### View Banned IPs

```bash
# List currently banned IPs
fail2ban-client status sshd

# Output example:
# Status for the jail: sshd
# |- Filter
# |  |- Currently failed: 0
# |  |- Total failed: 12
# |  `- File list: /var/log/auth.log
# `- Actions
#    |- Currently banned: 2
#    |- Total banned: 5
#    `- Banned IP list: 192.168.1.100 203.0.113.45
```

### Manual IP Management

```bash
# Ban an IP manually
fail2ban-client set sshd banip 192.168.1.100

# Unban an IP
fail2ban-client set sshd unbanip 192.168.1.100

# Unban all IPs
fail2ban-client unban --all
```

### View Logs

```bash
# Fail2ban logs
tail -f /var/log/fail2ban.log

# SSH authentication logs
tail -f /var/log/auth.log
```

## Advanced Configuration

### Whitelist Your IPs

```bash
# Edit jail.local to add trusted IPs
nano /etc/fail2ban/jail.local

# Add this to [DEFAULT] section:
# ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24 203.0.113.10
```

### Increase Security

```bash
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
# More aggressive settings
bantime = 3600      # 1 hour ban
findtime = 300      # 5 minute window
maxretry = 2        # Only 2 attempts allowed

[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 2
bantime = 7200      # 2 hour ban for SSH
EOF
```

## Troubleshooting

### Common Issues

1. **Fail2ban not starting:**
   ```bash
   # Check configuration syntax
   fail2ban-client -t
   
   # View detailed logs
   journalctl -u fail2ban -f
   ```

2. **Not blocking IPs:**
   ```bash
   # Verify log file exists
   ls -la /var/log/auth.log
   
   # Check if fail2ban can read logs
   fail2ban-client status sshd
   ```

3. **Locked yourself out:**
   ```bash
   # Access via console and unban your IP
   fail2ban-client set sshd unbanip YOUR_IP_ADDRESS
   ```

### Testing Fail2ban

```bash
# Generate test failures (from another machine)
ssh root@your-server-ip
# Enter wrong password 3 times

# Check if IP gets banned
fail2ban-client status sshd
```

## Monitoring and Maintenance

### Daily Monitoring Script

```bash
cat > /usr/local/bin/fail2ban-report.sh << 'EOF'
#!/bin/bash
echo "=== Fail2ban Daily Report ==="
echo "Date: $(date)"
echo
echo "Currently banned IPs:"
fail2ban-client status sshd | grep "Banned IP list"
echo
echo "Ban statistics:"
fail2ban-client status sshd | grep -E "Currently banned|Total banned|Total failed"
EOF

chmod +x /usr/local/bin/fail2ban-report.sh

# Add to cron for daily reports
echo "0 8 * * * /usr/local/bin/fail2ban-report.sh | mail -s 'Fail2ban Report' admin@yourdomain.com" | crontab -
```

### Log Rotation

```bash
# Ensure logs don't grow too large
cat > /etc/logrotate.d/fail2ban << 'EOF'
/var/log/fail2ban.log {
    weekly
    missingok
    rotate 4
    compress
    delaycompress
    postrotate
        systemctl reload fail2ban
    endscript
}
EOF
```

## Integration with Firewalls

### UFW Integration

```bash
# If using UFW, fail2ban integrates automatically
ufw enable
systemctl restart fail2ban
```

### Verify iptables Rules

```bash
# View current iptables rules created by fail2ban
iptables -L -n | grep f2b
```

## Performance Impact

- **CPU Usage:** Minimal (log monitoring)
- **Memory Usage:** ~10-20MB RAM
- **Network Impact:** None (only blocks, doesn't scan)
- **Log Impact:** Creates additional log entries

## Security Best Practices

✅ **Recommended Settings:**
- `maxretry = 2` (stricter than default)
- `bantime = 3600` (1 hour minimum)  
- `findtime = 300` (5 minute window)
- Whitelist your management IPs
- Enable email notifications
- Regular monitoring of banned IPs

❌ **Avoid:**
- Very short ban times (< 10 minutes)
- Too many retries (> 5)
- Not whitelisting your own IPs
- Ignoring fail2ban logs

---

**Success!** Your server now has automated intrusion prevention with fail2ban protecting against brute-force attacks.
