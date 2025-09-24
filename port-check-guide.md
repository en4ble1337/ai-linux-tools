# Port Connectivity Check Guide - Ubuntu 22.04

A step-by-step guide to verify NAT/firewall port mapping is configured correctly using Ubuntu 22.04 CLI.

## Prerequisites

- NAT/firewall configuration already set up on your router/firewall for the target ports
- Ubuntu 22.04 system with CLI access
- This example uses TCP port 49200

## Installation

Install netcat (network utility for reading/writing network connections):

```bash
sudo apt update && sudo apt install netcat-traditional -y
```

## Usage

### Step 1: Start Listening on the Port

Open a terminal and start netcat to listen on your target port:

```bash
nc -l -p 49200
```

> **Note**: This command will occupy the terminal session. The process continues running until terminated with `Ctrl+C`.

### Step 2: Verify Port is Listening

Open a new terminal session and verify the port is actively listening:

```bash
netstat -tuln | grep 49200
```

**Expected output:**
```
tcp        0      0 0.0.0.0:49200           0.0.0.0:*               LISTEN
```

#### Alternative Verification Commands

```bash
# Using ss (modern netstat replacement)
ss -tuln | grep 49200

# Using lsof (list open files)
sudo lsof -i :49200
```

### Step 3: External Port Testing

Test port accessibility from outside your network:

1. Determine your public IP address:
   ```bash
   curl ifconfig.me
   ```

2. Use an online port checker such as:
   - [portchecker.co](https://portchecker.co/)
   - [canyouseeme.org](https://canyouseeme.org/)

3. Enter your public IP address and port 49200
4. The result should show **open/accessible**

## Troubleshooting

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| Port shows as closed/filtered | Router NAT/firewall rules not configured | Verify port forwarding rules on router |
| Connection refused | netcat not running or crashed | Restart netcat listener |
| Can't determine public IP | Network connectivity issues | Check internet connection |

## Cleanup

When testing is complete, terminate the netcat listener:

```
Ctrl+C
```

## Security Considerations

- Only open ports that are necessary for your services
- Close test ports when not needed
- Regularly audit open ports and firewall rules
- Consider using non-standard ports for services when possible

## License

This guide is provided under the MIT License. Feel free to modify and distribute.
