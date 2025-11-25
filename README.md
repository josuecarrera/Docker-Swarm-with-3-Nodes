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

```

# This is the network config written by 'subiquity'
Modify the file to look like this. This sets the manager's static IP to 192.168.56.101.

```bash

network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.56.101/24
  version: 2

```

Apply the changes:

```bash

sudo netplan apply

```
Verify the IP: ip a show enp0s8 should now show 192.168.56.101.

### 4. Clone the VM to Create Workers

Now that you have a base VM, shut it down and clone it.

1. Right-click swarm-manager in VirtualBox and select Clone.

2.  Name: swarm-worker-1

3.  MAC Address Policy: Select Generate new MAC addresses for all network adapters. This is extremely important.

4. Choose "Full Clone" or "Linked Clone" (Linked is faster and saves space, but relies on the original).

Repeat the cloning process to create swarm-worker-2.

## Configure Worker VMs

Boot up swarm-worker-1 and swarm-worker-2 one at a time and make two small changes on each:

1. Change the hostname:

```bash

# On swarm-worker-1
sudo hostnamectl set-hostname swarm-worker-1
# On swarm-worker-2
sudo hostnamectl set-hostname swarm-worker-2

```

2. Change the static IP: Edit the netplan file (sudo nano /etc/netplan/00-installer-config.yaml) and change only the IP address.

    On swarm-worker-1: Use 192.168.56.102/24

    On swarm-worker-2: Use 192.168.56.103/24

    Apply the changes on each: sudo netplan apply

At this point, you should be able to ping all nodes from each other using their 192.168.56.x addresses. For example, from the manager: ping 192.168.56.102

## Step 3: Install Docker (On ALL 3 Nodes)

Run these commands on swarm-manager, swarm-worker-1, and swarm-worker-2.

1. Update packages:


```bash

sudo apt update
sudo apt install -y ca-certificates curl

```