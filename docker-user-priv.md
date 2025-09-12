### 🚀 Ubuntu User Setup Quick Guide

This guide provides a step-by-step process for creating a new user, granting them Docker access, and giving them `sudo` privileges on Ubuntu. This is especially useful for setting up development environments on servers or containers like LXC.

-----

### 1\. Create a New User and Set a Password

First, we'll create the user and immediately set their password. We'll name the user `node` for this guide.

```bash
# Create the user with a home directory and bash as the default shell
sudo useradd -m -s /bin/bash node

# Set the password for the new user
sudo passwd node
```

> **Note:** You'll be prompted to enter and confirm the new password. For this guide, we've used `node` as the password.

-----

### 2\. Add the User to the Docker Group

To allow the new user to run Docker commands without needing `sudo`, you must add them to the `docker` group.

```bash
# Add the 'node' user to the 'docker' group
sudo usermod -aG docker node
```

-----

### 3\. Grant the User Sudo Privileges

If you need the user to be able to run other administrative commands (like for scripts that include `sudo`), you must add them to the `sudo` group.

```bash
# Add the 'node' user to the 'sudo' group
sudo usermod -aG sudo node
```

-----

### 4\. Verify User Permissions

To confirm that the user has been added to the correct groups, use the `id` command. The output should show both `docker` and `sudo` in the list of groups.

```bash
# Check the user's group memberships
id node
```

**Expected Output:**

```
uid=1001(node) gid=1001(node) groups=1001(node),998(docker),27(sudo)
```

> **Tip:** You may need to log out and log back in as the new user for the group changes to take full effect.

-----

**Disclaimer:** Giving a user `sudo` access grants them full administrative control over the system. Only do this for trusted users and be mindful of security best practices.
