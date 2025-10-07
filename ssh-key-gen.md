# SSH Key Authentication Setup for LXC Container 

Complete step-by-step guide to set up SSH key authentication for root access to an Ubuntu 22.04 LXC container using PuTTY on Windows.

## Prerequisites

- Ubuntu 22.04 LXC container running
- Root password access enabled
- PuTTY suite installed on Windows (including PuTTYgen)
- Container IP address known

---

## Step 1: Generate SSH Key with PuTTYgen

1. **Open PuTTYgen** (comes with PuTTY suite)

2. **⚠️ IMPORTANT: Select the correct key type**
   - Select **RSA** (first option) - this is SSH-2 format
   - **DO NOT** select "SSH-1 (RSA)" - this is obsolete and will cause issues
   - Set key length to 2048 bits (default is fine)

3. **Generate the key:**
   - Click **"Generate"**
   - Move your mouse randomly in the blank area to generate randomness
   - Wait for key generation to complete

4. **Configure the key:**
   - Add a comment in **"Key comment"** field (e.g., `lxc-access-key`)
   - Optionally add a passphrase for extra security (leave blank for passwordless access)

5. **Save the keys:**
   - Click **"Save private key"** → save as `lxc-key.ppk` (PuTTY format)
   - Click **"Save public key"** → save as `lxc-key.pub`
   - **Copy the entire public key text** from the top text box (starts with `ssh-rsa AAAAB3NzaC1...`)
   - Keep PuTTYgen open or save this text to notepad - you'll need it in the next step

---

## Step 2: Install SSH Server (Keep Default Settings)

1. **Connect to your LXC container** (via console):
   ```bash
   lxc exec container-name bash
   ```

2. **Install SSH server:**
   ```bash
   apt update && apt install -y openssh-server
   systemctl enable ssh
   systemctl start ssh
   ```

3. **Verify SSH is running:**
   ```bash
   systemctl status ssh
   ```
   Should show "active (running)" in green

4. **Get your container's IP address:**
   ```bash
   ip addr show
   ```
   Note down the IP address (e.g., 10.x.x.x or 192.168.x.x)

---

## Step 3: Configure SSH for Root Access (Password Still Enabled)

**Run this single command to configure SSH properly for testing:**

```bash
cat << 'EOF' | bash
# Enable PasswordAuthentication for testing
sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Enable PermitRootLogin
sed -i 's/^#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/^PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# Enable PubkeyAuthentication
sed -i 's/^#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config

# Enable AuthorizedKeysFile
sed -i 's/^#AuthorizedKeysFile/AuthorizedKeysFile/' /etc/ssh/sshd_config

# Ensure PermitEmptyPasswords is disabled
sed -i 's/^#PermitEmptyPasswords no/PermitEmptyPasswords no/' /etc/ssh/sshd_config
sed -i 's/^PermitEmptyPasswords yes/PermitEmptyPasswords no/' /etc/ssh/sshd_config

# Test configuration
sshd -t

# If test passes, restart SSH
if [ $? -eq 0 ]; then
    systemctl restart ssh
    echo ""
    echo "✅ SSH configured successfully for testing"
    echo ""
    echo "Current settings:"
    grep -E "^PermitRootLogin|^PubkeyAuthentication|^PasswordAuthentication|^AuthorizedKeysFile" /etc/ssh/sshd_config
else
    echo "❌ Configuration test failed! SSH not restarted."
    exit 1
fi
EOF
```

**Expected output:**
```
✅ SSH configured successfully for testing

Current settings:
PermitRootLogin yes
PubkeyAuthentication yes
PasswordAuthentication yes
AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
```

---

## Step 4: Set Up SSH Key on LXC Container

1. **Create SSH directory for root:**
   ```bash
   mkdir -p /root/.ssh
   chmod 700 /root/.ssh
   ```

2. **Add the public key:**
   ```bash
   nano /root/.ssh/authorized_keys
   ```

3. **Paste your public key:**
   - Paste the public key text you copied from PuTTYgen
   - **Ensure it's all on ONE continuous line** (no line breaks in the middle)
   - Should look like: `ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ...long string... lxc-access-key`
   - Save and exit (`Ctrl+X`, `Y`, `Enter`)

4. **Set correct permissions:**
   ```bash
   chmod 600 /root/.ssh/authorized_keys
   chown root:root /root/.ssh/authorized_keys
   ```

5. **Verify the setup:**
   ```bash
   ls -la /root/.ssh/
   ```
   Should show:
   - `drwx------` (700) for .ssh directory
   - `-rw-------` (600) for authorized_keys file

   ```bash
   cat /root/.ssh/authorized_keys
   ```
   Should show your public key on one line starting with `ssh-rsa`

---

## Step 5: Configure PuTTY and TEST Key Authentication

**⚠️ CRITICAL: Keep your LXC console session open as backup!**

1. **Open PuTTY**

2. **Configure Session:**
   - Go to **Session** tab
   - Host Name: Enter your container's IP address
   - Port: 22
   - Connection type: SSH

3. **Configure SSH key authentication:**
   - Go to **Connection → SSH → Auth → Credentials**
   - Click **"Browse"** next to "Private key file for authentication"
   - Select your `lxc-key.ppk` file

4. **Set auto-login username:**
   - Go to **Connection → Data**
   - Enter `root` in **"Auto-login username"**

5. **Save session (optional but recommended):**
   - Go back to **Session** tab
   - Enter a name in **"Saved Sessions"** (e.g., `LXC-Root-Test`)
   - Click **"Save"**

6. **Test the connection:**
   - Click **"Open"**
   - If prompted about host key, click **"Accept"**
   - **You may be prompted for a password** - enter your root password
   - You should now be logged in as root

7. **Verify key is working:**
   
   Once connected, run this command:
   ```bash
   grep "Accepted publickey" /var/log/auth.log | tail -5
   ```
   
   If you see recent entries with "Accepted publickey for root", your key is working!

---

## Step 6: ⚠️ SECURE THE SERVER (Disable Password Authentication)

**ONLY proceed if Step 5 was successful and you confirmed the key works!**

### Option A: Automated Security Lockdown (Recommended)

Run this single command to disable password authentication:

```bash
cat << 'EOF' | bash
# Disable password authentication
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# Disable empty passwords
sed -i 's/#PermitEmptyPasswords no/PermitEmptyPasswords no/' /etc/ssh/sshd_config
sed -i 's/PermitEmptyPasswords yes/PermitEmptyPasswords no/' /etc/ssh/sshd_config

# Disable keyboard-interactive
sed -i 's/KbdInteractiveAuthentication yes/KbdInteractiveAuthentication no/' /etc/ssh/sshd_config
echo "KbdInteractiveAuthentication no" >> /etc/ssh/sshd_config

# Test and restart
sshd -t && systemctl restart ssh

echo ""
echo "✅ SSH is now secured - password authentication disabled"
echo "⚠️  Only SSH key authentication is allowed"
EOF
```

### Option B: Manual Security Configuration

1. **Edit SSH config again:**
   ```bash
   nano /etc/ssh/sshd_config
   ```

2. **Find and change these lines:**
   ```
   PasswordAuthentication no
   PermitEmptyPasswords no
   KbdInteractiveAuthentication no
   ```

3. **Save and exit** (`Ctrl+X`, `Y`, `Enter`)

4. **Test and restart:**
   ```bash
   sshd -t
   systemctl restart ssh
   ```

### Verify Security Configuration

```bash
grep -E "PasswordAuthentication|PubkeyAuthentication|PermitEmptyPasswords|KbdInteractiveAuthentication" /etc/ssh/sshd_config
```

**Expected output:**
```
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
KbdInteractiveAuthentication no
```

---

## Step 7: Final Test (Key-Only Authentication)

1. **Close your current PuTTY session**

2. **Open PuTTY and load your saved session** (or reconfigure)

3. **Click "Open"**

4. **You should now log in automatically without any password prompt**

5. **Success!** Your SSH is now secured with key-only authentication

---

## Troubleshooting

### Connection still asks for password after Step 6

**Possible causes:**

1. **Key file permissions wrong:**
   ```bash
   chmod 700 /root/.ssh
   chmod 600 /root/.ssh/authorized_keys
   ```

2. **Public key has line breaks:**
   ```bash
   cat /root/.ssh/authorized_keys
   ```
   Should be ONE continuous line. If broken across multiple lines:
   ```bash
   nano /root/.ssh/authorized_keys
   # Join all lines into one, save and exit
   ```

3. **Wrong key format in PuTTYgen:**
   - Make sure you selected **RSA** (SSH-2), not SSH-1
   - Regenerate the key if needed

4. **Check SSH logs for details:**
   ```bash
   tail -f /var/log/auth.log
   ```
   Try connecting in another window and watch for errors

### "Network error: Software caused connection abort"

**This error means authentication failed completely:**

1. **Re-enable password authentication temporarily:**
   ```bash
   sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
   systemctl restart ssh
   ```

2. **Verify your key is correct:**
   ```bash
   ssh-keygen -l -f /root/.ssh/authorized_keys
   ```
   Should show key fingerprint without errors

3. **Test again and check logs:**
   ```bash
   tail -f /var/log/auth.log
   ```

### "Access denied" or "Server refused our key"

1. **Verify PuTTY is using correct key file:**
   - Check Connection → SSH → Auth → Credentials
   - Make sure it points to your `.ppk` file (not `.pub`)

2. **Verify auto-login username is set:**
   - Connection → Data
   - Should say `root`

3. **Check SELinux/AppArmor (if applicable):**
   ```bash
   # Check if SELinux is enforcing
   getenforce
   
   # Check SSH in audit log
   grep sshd /var/log/audit/audit.log | tail -20
   ```

### Locked Out?

If you disabled passwords and can't get in with your key:

```bash
# From Proxmox/host console:
lxc exec container-name bash

# Re-enable password authentication
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl restart ssh
```

---

## Security Considerations

✅ **Current Security Features (After Step 6):**

- **Key-only authentication:** Password login is completely disabled
- **No empty passwords:** Extra protection layer
- **No keyboard-interactive:** Prevents bypass methods
- **Root access controlled:** Only accessible via SSH key

⚠️ **Important Security Notes:**

- **Anyone with the private key** can access your container as root
- **Keep your private key secure** and never share it
- Consider adding a passphrase to the private key for extra security

📚 **For Production Environments, Consider:**

- Using a regular user account instead of root (with sudo)
- Adding a strong passphrase to the private key
- Implementing IP address restrictions in sshd_config
- Using certificate-based authentication
- Setting up fail2ban for brute-force protection
- Regular security audits and key rotation

---

## Alternative: Ed25519 Keys (More Secure)

For enhanced security, you can use **Ed25519** instead of RSA:

1. In PuTTYgen, select **"EdDSA"**
2. Set curve to **"Ed25519"**
3. Generate and follow the same steps as above
4. The public key will start with `ssh-ed25519` instead of `ssh-rsa`
5. Ed25519 keys are smaller, faster, and more secure than RSA

---

## Quick Reference: File Locations and Permissions

```
/root/.ssh/                    → drwx------ (700)
/root/.ssh/authorized_keys     → -rw------- (600)
/etc/ssh/sshd_config           → Main SSH configuration
/var/log/auth.log              → SSH authentication logs
```

---

**Success!** You now have secure, passwordless SSH key authentication set up for your LXC container.
