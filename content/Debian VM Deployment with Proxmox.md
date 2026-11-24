---
publish: true
created: 2026-11-23T23:33:50.948Z
modified: 2026-11-24T22:25:25.953Z
tags:
  - proxmox
  - debian
  - rebuild
  - homelab
  - opnsense
---

**A guide for the deployment of Debian virtual machines with Proxmox**

This is third part in my homelab infrastructure rebuild project, [[Homelab Infrastructure Rebuild Overview]]. This guide assumes you have a Proxmox instance running with OPNsense. See [[Migrating to Proxmox]] for Proxmox installation and configuration instructions, and [[OPNsense Firewall Deployment, Configuration, and Segmentation with Proxmox]] for the OPNsense configuration. I use Debian for my Docker hosts, and highly recommend it for its stability, light resource usage, and lack of bloat. I will discuss the configuration of Docker on Debian in a future guide. While this guide is written primarily for Debian as a Docker host, the Proxmox deployment portion of this guide is highly transferable to other Linux distros.

# Adding the Debian ISO to Proxmox

We need to add a Debian ISO to deploy from to Proxmox. To do this, download the [ISO](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/) file to your work computer. Then navigate to your Proxmox management GUI:

1. Navigate to `<your-proxmox-node>` | local (`<your-proxmox-node>`) | ISO Images
2. Click **Upload**
   1. Alternatively click **Download from URL** if you have an ISO download URL
3. Click **Select File**
4. Select `<your-debian-iso>`
5. Click **Upload**

# Deploying the Debian VM

1. Click **Create VM**
2. Configure to the table:

| Option                          | Setting               | Section | Note                                                                                                                                       |
| ------------------------------- | --------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Node                            | `<your-proxmox-node>` | General |                                                                                                                                            |
| VM ID                           | `<your-vm-id>`        |         | Can be any number. I deploy each VM id with an increment of 10. So if you have a VM of ID 100, do 110. Then 120 for the next VM you deploy |
| Name                            | `<your-vm-name>`      |         | I name each vm after its purpose. If this is a VM for the DMZ, I would call it `dmz`                                                       |
| Start at boot (Advanced)        | Yes                   |         | Optional, but if you're using this to host you probably want it to start at boot                                                           |
| Start/Shutdown order (Advanced) | Any                   |         |                                                                                                                                            |
| Use CD/DVD disc image file      | Yes                   | OS      |                                                                                                                                            |
| Storage                         | local                 |         |                                                                                                                                            |
| ISO image                       | `<your-debian-iso>`   |         |                                                                                                                                            |
| Guest OS Type                   | Linux                 |         |                                                                                                                                            |
| Graphic card                    | Default               | System  |                                                                                                                                            |
| Machine                         | q35                   |         |                                                                                                                                            |
| BIOS                            | OVMF (UEFI)           |         | UEFI allows iGPU passthrough                                                                                                               |
| SCSI controller                 | VirtIO SCSI Single    |         |                                                                                                                                            |
| Qemu Agent                      | Yes                   |         | Optional but highly recommended agent for Proxmox                                                                                          |
| Add TPM                         | No                    |         |                                                                                                                                            |
| Bus/Device                      | SCSI                  | Disks   |                                                                                                                                            |
| Storage                         | local-lvm             |         |                                                                                                                                            |
| Disk size                       | 64 GB                 |         | Can be whatever size you want, I recommend at least 64GB for a Docker host                                                                 |
| Cache                           | Default               |         |                                                                                                                                            |
| Discard                         | Yes                   |         |                                                                                                                                            |
| IO thread                       | Yes                   |         |                                                                                                                                            |
| Sockets                         | 1                     | CPU     |                                                                                                                                            |
| Cores                           | 4                     |         | Choose what you feel you need, I recommend 4 cores for a Docker host                                                                       |
| Type                            | host                  |         |                                                                                                                                            |
| Memory                          | 8192MB                | Memory  | Can be whatever size you want, I recommend 8GB for a Docker host                                                                           |
| Ballooning Device (Advanced)    | No                    |         |                                                                                                                                            |
| Bridge                          | vmbr1                 | Network |                                                                                                                                            |
| VLAN Tag                        | `<your-vlan-tag>`     |         | Choose the appropriate VLAN tag for the subnet you are deploying this on                                                                   |
| Firewall                        | No                    |         |                                                                                                                                            |
| Model                           | VirtIO                |         |                                                                                                                                            |
| MAC Address                     | Leave blank           |         |                                                                                                                                            |
3\. Click **Finish**

# Debian VM Installation

The rest of this guide requires your VM to have internet access. If you are following my guides currently, your VM will not have internet access. The next guide I post will contain an in-depth look at OPNsense firewall rule configuration. In the meantime, navigate to OPNsense and do the following to allow internet access while keeping the segmentation:

1. Navigate to Firewall | Aliases
2. Click **+**
3. Configure to the table

| Enabled | Name    | Type       | Categories  | Content                                   | Statistics | Description      |
| ------- | ------- | ---------- | ----------- | ----------------------------------------- | ---------- | ---------------- |
| Yes     | RFC1918 | Network(s) | Leave blank | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 | No         | RFC1918 grouping |

4. Click **Save**
5. Click **Apply**
6. Navigate to Firewall | Rules
7. Click **+**
8. Configure according to the table:

| Enabled | Categories  | Description                    | Invert Interface | Interface               | Quick | Action | Direction | Version | Protocol | Invert Source | Source                          | Source Port | Invert Destination | Destination   | Destination Port | Log | Gateway |
| ------- | ----------- | ------------------------------ | ---------------- | ----------------------- | ----- | ------ | --------- | ------- | -------- | ------------- | ------------------------------- | ----------- | ------------------ | ------------- | ---------------- | --- | ------- |
| Yes     | Leave blank | `<your-machine>` Internet Only | No               | `<your-vlan-interface>` | Yes   | Pass   | In        | IPv4    | Any      | No            | `<your-vlan-interface>` Network | Any         | Yes                | RFC1918 Alias | Any              | Yes | None    |

9. Click **Save**
10. Click **Apply**

We now want to install and configure Debian. To do so access your Proxmox management GUI:

1. Navigate to `<your-proxmox-node` | `<your-deployed-debian-vm>` | Console
2. Click **Start Now**
3. Select _Graphical install_
4. Press **Enter**
5. Select `<your-chosen-language>`
6. Click **Continue**
7. select `<your-country>`
8. click **Continue**
9. select `<your-keyboard-layout>`
10. click **Continue** | **Continue**
11. Select _Configure network manually_
12. Click **Continue**
13. Enter `<your-machine-ip>`/24
    1. For the first 3 octets of the IP, use the subnet that your assigned VLAN uses, and for the last octet choose a number under 255. For example, if your VLAN 40 uses the 10.0.40.0/24 subnet, you can use 10.0.40.10/24.
14. Enter `<your-subnet-gateway-address>`
    1. For this step and the following, it is typically going to be the first address of your subnet. For example, if you're using the 10.0.40.0/24 subnet, 10.0.40.1 is what you would put here.
15. Enter `<your-subnet-gateway-address>`
16. Enter `<your-hostname>`
    1. Typically the same as what you named your VM
17. Enter `<your-home-domain>`
18. Click **Continue**
    1. Skip setting up the root account here
19. Enter `<your-name>`
20. Enter `<your-username>`
21. Type `<your-password>` in both boxes
22. Click **Continue**
23. Select `<your-timezone>`
24. Click **Continue**
25. Select _Guided - use entire disk_
26. Click **Continue**
27. Select `<your-disk>`
28. Click **Continue**
29. Select _All files in one partition_
30. Click **Continue** | **Continue**
31. Select _Yes_
32. Click **Continue**
33. Select _No_
34. Click **Continue**
35. Select `<your-country>`
36. Click **Continue**
37. Select _deb.debian.org_
38. Click **Continue** | **Continue**
39. Select _No_
40. Click **Continue**
41. Untick _GNOME_
42. Untick _Debian Desktop Environment_
43. Tick _SSH server_
44. Tick _standard system utilities_
45. Click **Continue** | **Continue**

Now you should have a working Debian VM. You'll also want to unmount the  iso in Proxmox:

1. Navigate to `<your-proxmox-node` | `<your-deployed-debian-vm>` | Hardware
2. Select _CD/DVD Drive_
3. Click **Edit**
4. Tick _Do not use any media_
5. Click **OK**

# Debian VM configuration

Lastly, we want to install some basic things and ensure our Debian is up to date. We'll be installing and configuring the qemu guest agent, the purpose of which is explained [[OPNsense Firewall Deployment, Configuration, and Segmentation with Proxmox#Qemu Guest Agent Installation|here]], curl for downloads, as well as ca-certificates which allows for https downloads. In Proxmox:

1. Navigate to `<your-proxmox-node` | `<your-deployed-debian-vm>` | Console
2. Login with your username and password
3. Enter the following command:
   1. This may require a VM restart, do so, and then repeat steps 1 and 2 and move to step 4

```
sudo apt update && sudo apt full-upgrade -y
```

4. Enter the following command

```
sudo apt install -y qemu-guest-agent curl ca-certificates
```

Now you have deployed Debian VM that is ready to serve as a docker host, or any other type of host.

# Closing Notes

Now that you have a Debian VM setup, we'll need to get firewall rules configured for it so it can securely provide services, like SSH. My next guide will cover OPNsense firewall rules in depth, with an explanation on how to write your own as well as some recommended rules.
