# A Quick Guide to Nmap 🗺️

Nmap (Network Mapper) is an essential open-source tool for network exploration and security auditing. It allows you to discover which hosts are available on a network, what services those hosts are offering, what operating systems they're running, and more.

-----

## Installation

You can install `nmap` on most operating systems using their native package managers.

### Linux (Debian/Ubuntu)

```bash
sudo apt update
sudo apt install nmap
```

### Linux (RHEL/CentOS/Fedora)

```bash
sudo dnf install nmap
# Or for older systems
sudo yum install nmap
```

### macOS (using Homebrew)

```bash
brew install nmap
```

### Windows

Download the official installer from the [nmap.org download page](https://nmap.org/download.html).

-----

## Basic Usage & Examples

The basic syntax for `nmap` is `nmap [options] [target]`.

A common and important option is `-Pn`, which tells `nmap` to skip the initial ping check. This is useful when scanning hosts that block pings, which would otherwise cause `nmap` to report the host as down.

1.  **Scan a Single Port**
    Use the `-p` flag to specify a single port to scan.

    *Example: Scan for port 22 (SSH) on the target `192.168.1.1`.*

    ```bash
    nmap -p 22 192.168.1.1
    ```

2.  **Scan Multiple Specific Ports**
    You can scan a list of ports by separating them with commas.

    *Example: Scan for ports 80 (HTTP), 443 (HTTPS), and 8080.*

    ```bash
    nmap -p 80,443,8080 192.168.1.1
    ```

3.  **Scan a Range of Ports**
    To scan a contiguous range of ports, use a hyphen.

    *Example: Scan all ports from 1 to 1000.*

    ```bash
    nmap -p 1-1000 192.168.1.1
    ```

4.  **Scan a Host That Blocks Pings**
    Combine the `-Pn` flag with your port scan to ensure it runs even if the host doesn't respond to pings.

    *Example: Scan for port 3389 (RDP) on a host that might be blocking pings.*

    ```bash
    nmap -Pn -p 3389 192.168.1.1
    ```

-----

## Interpreting the Output

The most important piece of information in the output is the port's **STATE**.

  * **open**: The port is open, and an application is actively accepting connections. ✅
  * **closed**: The port is accessible, but there is no application listening on it. ❌
  * **filtered**: A firewall or other network obstacle is blocking the port, so `nmap` cannot determine if it's open or closed. 🛡️
