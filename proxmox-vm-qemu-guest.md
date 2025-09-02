# 🧩 QEMU Guest Agent Installation on Ubuntu 22.04 (Proxmox VM)

Enable enhanced VM communication (e.g. IP reporting, shutdown, freeze/thaw) by installing the QEMU Guest Agent inside your Ubuntu guest and configuring it on the Proxmox host.

---

## 📦 Step 1: Install QEMU Guest Agent in Ubuntu 22.04

Run the following commands inside the guest VM:

```bash
sudo apt update
sudo apt install -y qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent
```

This installs the agent, enables it to start on boot, and starts it immediately.

---

## 🖥️ Step 2: Enable Guest Agent in Proxmox Host

Run this on the Proxmox host to enable agent support for the VM:

```bash
qm set <VMID> --agent enabled=1
```

Replace `<VMID>` with your actual VM ID. To list VMs:

```bash
qm list
```

---

## ✅ Step 3: Verify Agent Communication

Test the agent connection from the Proxmox host:

```bash
qm agent <VMID> ping
```

Expected output:

```json
{"return":""}
```

If you see this, the agent is working correctly.

---

## 🧪 Optional: Query Guest Info

Once the agent is active, you can retrieve guest details:

```bash
qm agent <VMID> get-osinfo
qm agent <VMID> network-get-interfaces
```

---

## 🧰 Automation Tip

To automate this during VM provisioning, embed the guest install into your cloud-init or post-deploy script, and wrap the `qm set` into your VM creation workflow.

---

Let me know if you want this templated for multiple VMs or integrated into a bash provisioning script. I can help you streamline it even further.
