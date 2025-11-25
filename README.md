# How to Set Up a 3-Node Docker Swarm in VirtualBox

This guide outlines the step-by-step process to set up a Docker Swarm cluster using Oracle VirtualBox and Ubuntu Server. We will configure one **Manager** node and two **Worker** nodes.

## 📋 Prerequisites

* **Oracle VirtualBox** installed on your host machine.
* **Ubuntu Server 22.04 LTS ISO** (or a similar Linux distribution).
* Basic knowledge of the Linux command line.

## 🏗️ Architecture

We will create 3 Virtual Machines with the following configuration:

| Hostname | Role | IP Address (Host-Only) | RAM | vCPU |
| :--- | :--- | :--- | :--- | :--- |
| `swarm-manager` | Manager | **192.168.56.101** | 2GB+ | 2 |
| `swarm-worker-1` | Worker | **192.168.56.102** | 1GB+ | 1 |
| `swarm-worker-2` | Worker | **192.168.56.103** | 1GB+ | 1 |

---

## Step 1: VirtualBox Network Configuration

Before creating VMs, create a private network for node-to-node communication.

1.  Open VirtualBox.
2.  Go to **File > Tools > Network Manager**.
3.  Create a **Host-Only Network** (usually named `vboxnet0`).
4.  Ensure the IPv4 Address is `192.168.56.1` and Mask is `255.255.255.0`.

---

## Step 2: Create the Base VM (Manager)

1.  Create a new VM named `swarm-manager`.
2.  Install Ubuntu Server.
    * **Hostname:** `swarm-manager`
    * **User:** `docker-admin` (or your preference)
    * **SSH:** Select "Install OpenSSH Server" during setup.
3.  **Network Settings (Crucial):**
    * **Adapter 1:** NAT (For Internet access).
    * **Adapter 2:** Host-only Adapter (Select `vboxnet0`).

### Configure Static IP
Log into the VM and configure the Host-Only interface (`enp0s8` usually).

```bash
sudo nano /etc/netplan/00-installer-config.yaml

Modify the file to look like this. This sets the manager's static IP to 192.168.56.101.


```bash
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.56.101/24
  version: 2
