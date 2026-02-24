# NFS Server Configuration on RHEL — Complete Project Guide

## 📌 Project Overview

In this project, I configured an NFS (Network File System) server on RHEL and connected a client system to access the shared directory. The goal was to create a reliable and secure file-sharing setup inside a Linux environment.
This documentation reflects the exact process I followed, including installation, configuration, testing, and best practices.

---

## 🎯 Objectives

* Install and configure NFS server on RHEL
* Export a directory to client machines
* Mount the NFS share on the client
* Verify read/write functionality

---

## 🧱 Lab Environment

| Role       | Hostname   | IP Address   | OS   |
| ---------- | ---------- | ------------ | ---- |
| NFS Server | nfs-server | 192.168.1.10 | RHEL |
| NFS Client | nfs-client | 192.168.1.20 | RHEL |

> Note: Replace IPs according to your environment.

---

# 🔧 Step 1: Install Required Packages (Server)

First, I installed the NFS utilities on the server.

```bash
dnf install -y nfs-utils
```

---

# 🚀 Step 2: Enable and Start NFS Services

Then I enabled and started the required services.

```bash
systemctl enable --now nfs-server
systemctl enable --now rpcbind
```

To verify:

```bash
systemctl status nfs-server
```

---

# 📁 Step 3: Create Shared Directory

Next, I created the directory that I wanted to share.

```bash
mkdir -p /nfsshare
chmod 755 /nfsshare
```

> In production, always use proper permissions instead of 777.

---

# ⚙️ Step 4: Configure Exports

I edited the exports file:

```bash
vi /etc/exports
```

Then I added:

```bash
/nfsshare 192.168.1.0/24(rw,sync,root_squash)
```

### 🔍 Explanation of Options

* **rw** → allows read and write access
* **sync** → ensures data is written safely to disk
* **root_squash** → maps client root user to nfsnobody for security

---

### 🌐 Understanding the Client IP Concept

In the exports entry:

```bash
/nfsshare 192.168.1.0/24(rw,sync,root_squash)
```

The IP or network specified here represents **which client machines are allowed to access the NFS share**, not the server IP.

#### Why I used `192.168.1.0/24`

I used this subnet because my client systems were in the same network range. By specifying the subnet, I allowed all machines within that network to access the share.

#### ✅ Allow only one specific client

If I want to allow access to just one machine, I can specify its exact IP:

```bash
/nfsshare 192.168.1.20(rw,sync,root_squash)
```

This ensures only that client can mount the share.

#### ✅ Allow an entire subnet

To allow all machines in a network:

```bash
/nfsshare 192.168.1.0/24(rw,sync,root_squash)
```

This is useful in controlled internal networks.

#### ⚠️ Allow all clients (not recommended for production)

If I use:

```bash
/nfsshare *(rw,sync,root_squash)
```

It means **any machine** that can reach the server may try to mount the share. This is generally avoided in production due to security risks.

**Best practice:** Always restrict access to specific IPs or subnets whenever possible.

---

# 🔄 Step 5: Apply Export Configuration

After saving the file, I applied the configuration.

```bash
exportfs -rav
```

To verify:

```bash
exportfs -v
```

---

# 🔥 Step 6: Configure Firewall

Since the firewall was enabled, I allowed NFS services.

```bash
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload
```

---

## 🌐 NFS Network Ports and Communication Flow

During this project, I also reviewed the network ports used by NFS to ensure proper firewall configuration and troubleshooting capability.

### 🔢 Important NFS Ports

| Service | Port    | Protocol |
| ------- | ------- | -------- |
| NFS     | 2049    | TCP/UDP  |
| RPCBind | 111     | TCP/UDP  |
| Mountd  | dynamic | TCP/UDP  |

### 🧭 How NFS Communication Works

When a client mounts an NFS share, the communication happens in this sequence:

1. Client contacts **RPCBind (port 111)** to discover NFS services
2. RPCBind provides the mountd port
3. Client communicates with **mountd**
4. Finally, data transfer happens over **NFS (port 2049)**

This is why firewall configuration must allow all required NFS-related services.

### 🔍 Verification Commands I Used

To verify listening ports on the server:

```bash
netstat -tunlp | grep -E '2049|111'
```
---

---

# 💻 Client Configuration

Now I moved to the client machine.

---

## Step 7: Install NFS Utilities (Client)

```bash
dnf install -y nfs-utils
```

---

## Step 8: Verify Available Exports

From the client, I verified the server exports.

```bash
showmount -e 192.168.1.10
```

Expected output should list `/nfsshare`.

---

## Step 9: Create Mount Point

```bash
mkdir -p /mnt/nfsclient
```

---

## Step 10: Mount the NFS Share

```bash
mount -t nfs 192.168.1.10:/nfsshare /mnt/nfsclient
```

Verification:

```bash
df -Th
```

---

## Step 11: Permanent Mount (fstab)

To make the mount persistent after reboot, I edited:

```bash
vi /etc/fstab
```

Added:

```bash
192.168.1.10:/nfsshare   /mnt/nfsclient   nfs   defaults   0 0
```

After updating fstab, I reloaded systemd and tested the mount.

```bash
systemctl daemon-reload
mount -a
```


---

# 🧪 Testing the Setup

## On Server

```bash
touch /nfsshare/server_test.txt
````

## On Client

```bash
ls /mnt/nfsclient
```

If the file is visible, the setup is working correctly.

I also tested write access from the client:

```bash
touch /mnt/nfsclient/client_test.txt
```

---

# 🚨 Troubleshooting I Considered

During the setup, these were the common checks:

* Verified network connectivity using `ping`
* Checked firewall rules
* Confirmed export entries
* Ensured services were running
* Verified SELinux settings

---

# 🔐 Security Best Practices

While configuring, I followed these recommendations:

* Avoided using `no_root_squash` in production
* Limited access to specific subnet
* Used proper directory permissions
* Kept firewall enabled with required ports only
* Verified SELinux policy instead of disabling it

---

# 📊 Project Outcome

By the end of this project:

* NFS server was successfully configured
* Client system could mount the share
* Read/write operations were working
* Security settings were properly applied
* Persistent mount was configured

---

# 🧠 What I Learned

Through this hands-on setup, I gained practical experience in:

* Linux file sharing using NFS
* Service management with systemctl
* Network-based access control
* Firewall and SELinux integration
* Persistent filesystem mounts

This project strengthened my understanding of Linux storage networking in real-world scenarios.

---

# ✅ Conclusion

In this project, I successfully deployed and tested an NFS-based file sharing solution on RHEL. The setup proved reliable for internal network file sharing and demonstrated how proper configuration of exports, firewall, and SELinux ensures both functionality and security.
This implementation can be extended further using automount, Kerberos authentication, or high-availability storage depending on enterprise requirements.

---
