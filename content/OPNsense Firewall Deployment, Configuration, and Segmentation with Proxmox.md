---
publish: true
created: 2026-11-11T18:16:21.463Z
modified: 2026-11-12T00:31:41.955Z
tags:
  - rebuild
  - homelab
  - proxmox
  - opnsense
---

**A guide for the deployment and configuration of a virtualized OPNsense firewall for segmentation with Proxmox behind a double NAT.**

This is the second part of my homelab "rebuild" project, [[Homelab Infrastrcture Rebuild Overview]]
In my homelab configuration, my services are behind a double NAT, meaning that there is another router in front of OPNsense. If using OPNsense as your router, some configurations may differ. Keep in mind that the network behind your router, is the WAN side in the context of OPNsense.

# Virtual Switch Creation in Proxmox

The first thing you'll need to do before your deployment of an OPNsense VM is create a virtual  trunked switch to manage your VLANs. To do this, open your Proxmox management GUI:

1. Navigate to `<your-proxmox-node>` | System | Network
2. Click **Create** | **Linux Bridge**
3. Type `<your-virtual-trunked-switch-name>` in _Name:_
4. Tick the _VLAN aware:_ checkbox
5. Click **Create**
6. Click **Apply Configuration**

# OPNsense Virtual Machine Deployment

In order to deploy an OPNsense VM, you'll need to provide Proxmox the OPNsense ISO. Go to the [OPNsense download page](https://opnsense.org/download/), select the "dvd" image type, and download the ISO. To upload the ISO to proxmox:

1. Navigate to `<your-proxmox-node>` | local (`<your-proxmox-node>`) | ISO Images
2. Click **Upload**
   1. Alternatively click **Download from URL** if you have an ISO download URL
3. Click **Select File**
4. Select `<your-opnsense-iso>`
5. Click **Upload**

Now you can deploy your OPNsense VM:

1. Click **Create VM**
2. Configure to the table:

| Option                       | Setting               | Section |
| ---------------------------- | --------------------- | ------- |
| Node                         | `<your-proxmox-node>` | General |
| VM ID                        | 100                   |         |
| Name                         | opnsense              |         |
| Use CD/DVD disc image file   | Yes                   | OS      |
| Storage                      | local                 |         |
| ISO image                    | `<your-opnsense-iso>` |         |
| Guest OS Type                | Other                 |         |
| Graphic card                 | Default               | System  |
| Machine                      | q35                   |         |
| BIOS                         | SeaBIOS               |         |
| SCSI controller              | VirtIO SCSI Single    |         |
| Qemu Agent                   | Yes                   |         |
| Add TPM                      | No                    |         |
| Bus/Device                   | SCSI                  | Disks   |
| Storage                      | local-lvm             |         |
| Disk size                    | 32 GB                 |         |
| Cache                        | Default               |         |
| Discard                      | Yes                   |         |
| IO thread                    | Yes                   |         |
| Sockets                      | 1                     | CPU     |
| Cores                        | 2                     |         |
| Type                         | host                  |         |
| Memory                       | 4096 MB               | Memory  |
| Ballooning Device (Advanced) | No                    |         |
| Bridge                       | vmbr0                 | Network |
| VLAN Tag                     | Leave blank           |         |
| Firewall                     | No                    |         |
| Model                        | VirtIO                |         |
| MAC Address                  | Leave blank           |         |

3. Click **Finish**

Now you'll need to add your virtual trunked switch to the VM.

1. Navigate to 100 (opnsense) | Hardware
2. Click **Add** | **Network Device**
3. Configure to the table:

| Option      | Setting                              |
| ----------- | ------------------------------------ |
| Bridge      | `<your-virtual-trunked-switch-name>` |
| VLAN Tag    | Leave blank                          |
| Firewall    | No                                   |
| Model       | VirtIO                               |
| MAC Address | Leave blank                          |

4. Click **Add**

# OPNsense Installation

Now that you have an OPNsense VM created and configured, you'll need to install OPNsense using your mounted ISO. In Proxmox:

1. Navigate to 100 (opnsense) | Console
2. Click **Start**
   1. For the rest of the steps, complete these from the console within Proxmox
3. Enter `installer` for _login:_
4. Enter `opnsense` for _Password:_
5. Select `<your-keyboard-layout>`
6. Press **ENTER**
7. Select _Install (UFS)_
8. Press **ENTER**
9. Select _da0_
10. Press **ENTER** | **ENTER**
11. Once installed, select _Change root password_
12. Press **ENTER**
13. Enter `<your-opnsense-password>`
14. Enter `<your-opnsense-password>` again to confirm
15. Select _Complete Install_
16. Press **ENTER**
17. Select _Reboot Now_
18. Press **Enter**

# OPNsense Configuration

Now you have OPNsense installed on a Proxmox virtual machine. You need to configure it next. In Proxmox:

1. Navigate to 100 (opnsense) | Console1. Navigate to 100 (opnsense) | Console
2. Enter `root` for _login:_
3. Enter `<your-opnsense-password>` for _Password:_
4. Enter `1`
5. Configure according to the table:

| Option           | Setting                            |
| ---------------- | ---------------------------------- |
| Configure LAGGs? | No                                 |
| Configure VLANs? | No (You will do this from the GUI) |
| WAN              | vtnet0                             |
| LAN              | vtnet1                             |

6. Press **ENTER**
7. Enter `Y` | `2` | `1`
8. Configure vtnet1 to the table:

| Option                             | Setting                                                 |
| ---------------------------------- | ------------------------------------------------------- |
| DHCP                               | No                                                      |
| IPv4                               | `<your-lan-ip>`, something like 192.168.1.1 or 10.0.0.1 |
| Subnet bits                        | 24                                                      |
| Upstream Gateway                   | None                                                    |
| IPv6 via WAN tracking              | No (Yes if not using double NAT)                        |
| DHCP6                              | No                                                      |
| IPv6                               | None                                                    |
| DHCP Server                        | No (Probably Yes with a range if not using double NAT)  |
| Revert to HTTP                     | No                                                      |
| Generate a self-signed certificate | Yes                                                     |
| Web GUI defaults                   | Yes                                                     |
If you're doing the double NAT, you'll also want to reserve an IP address for OPNsense in your router. On default, OPNsense blocks GUI access from the WAN side, this is important for the double NAT. Because your devices are on the WAN side in regards to OPNsense, you'll need to temporarily disable them.
In your OPNsense console, from Proxmox:

1. Enter `8`
2. Enter `pfctl -d`

Now you can access OPNsense from a WAN side device. To finish configuration of OPNsense from the GUI:

1. Navigate to `<your-opnsense-wan-ip>` in a web browser
2. Type `root` _Username:_
3. Type `<your-opnsense-password>` for _Password:_
4. Click **Login**
5. Configure the wizard to the table:

| Option                          | Setting                                                        | Section             |
| ------------------------------- | -------------------------------------------------------------- | ------------------- |
| Hostname                        | opnsense                                                       | General Information |
| Domain                          | `<your-home-domain>`, could be something like home.example.com |                     |
| Language                        | English                                                        |                     |
| Timezone                        | Etc/UTC                                                        |                     |
| DNS Servers                     | `<your-first-router-ip>`                                       |                     |
| Enable Resolver                 | Yes                                                            |                     |
| Enable DNSSEC Support           | Yes                                                            |                     |
| Harden DNSSEC data              | Yes                                                            |                     |
| Disable WAN                     | No                                                             | Network (WAN)       |
| Type                            | DHCP                                                           |                     |
| MAC (spoofed)                   | Leave blank                                                    |                     |
| MTU                             | Leave blank                                                    |                     |
| MSS                             | Leave blank                                                    |                     |
| DHCP hostname                   | Leave blank                                                    |                     |
| Block RFC1918 Private Networks  | No (Yes if not using double NAT)                               |                     |
| Blog bogon networks             | No (Yes if not using double NAT)                               |                     |
| Disable LAN                     | No                                                             | Network (LAN)       |
| IP Address                      | `<your-lan-ip>`                                                |                     |
| Configure DHCP Server           | No (Probably Yes if not using double NAT)                      |                     |
| Optimize for Multiwan           | No                                                             | Deployment Type     |
| Automatic DHCP/DNS Registration | No (Probably Yes if not using a double NAT)                    |                     |
| Optimize for IPsec              | No                                                             |                     |
When prompted to set an initial password, leave it blank as we already set one when installing OPNsense.
6\. Click **Apply**

In the context of a double NAT, while the Block RFC1918 Private Networks rule stops private addresses from being dropped, the default state of your WAN side is to deny all incoming traffic, you need to create a temporary firewall rule to allow incoming traffic from the device you are using to configure OPNsense.
The reason this is temporary is that we are going to create a VLAN specifically for management, which will be accessible from a VPN or trusted devices on their own VLAN. Proxmox and OPNsense will only be manageable from those connections, as they will have their management interfaces moved to the management VLAN. We will also further segment our external services into its own VLAN and machine, a DMZ, and our internal services into its own VLAN and machine. These will also only be manageable from the VPN or trusted devices.
To create the Firewall rule, you likely will need to disable firewall rules again. Refer to the above step to do that again. Once you can access the OPNsense management GUI:

1. Navigate to Firewall | Rules
2. Click **+** Icon
3. Configure to the table

| Enabled | Categories  | Description                                 | Invert Interface | Interface | Quick | Action | Direction | Version | Protocol | Invert Source | Source                                           | Source Port | Invert Destination | Destination   | Destination Port | Log | Gateway |
| ------- | ----------- | ------------------------------------------- | ---------------- | --------- | ----- | ------ | --------- | ------- | -------- | ------------- | ------------------------------------------------ | ----------- | ------------------ | ------------- | ---------------- | --- | ------- |
| Yes     | Leave blank | Temporary - OPNsense access from trusted PC | No               | WAN       | Yes   | Pass   | In        | IPv4    | TCP      | No            | Single host or network `<your-client-device-ip>` | Any         | No                 | This Firewall | HTTPS (443)      | Yes | None    |

1. Click **Save**
2. Click **Apply**

Now, we can configure our VLANs for segmentation.

# OPNsense VLAN Configuration

Our VLANs will serve the segmentation of your network, allowing for granular management of network traffic. In your OPNsense management GUI:

1. Navigate to Interfaces | Devices | VLAN
2. Click **+**
3. Create VLANs according to the table, click **Save** after you fill each out.

| Device      | Parent | VLAN Tag | VLAN priority            | Description |
| ----------- | ------ | -------- | ------------------------ | ----------- |
| Leave blank | vtnet1 | 10       | Best Effort (0, default) | MANAGEMENT  |
| Leave blank | vtnet1 | 20       | Best Effort (0, default) | TRUSTED     |
| Leave blank | vtnet1 | 30       | Best Effort (0, default) | INTERNAL    |
| Leave blank | vtnet1 | 40       | Best Effort (0, default) | DMZ         |
| Leave blank | vtnet1 | 50       | Best Effort (0, default) | STORAGE     |

4. After all VLANs are created, click **Apply**

Each VLAN serves a different purpose. 10 is for managing our main services, like Proxmox. 20 is for our trusted devices, which will be able to manage all of our homelab services. 30 is for internal services, so only services that will not be publicly available. 40 is our DMZ, for publicly available services. 50 is for storage devices, such as NAS.

Now, we need to assign these VLANs network interfaces.

1. Navigate to Interfaces | Assignments
2. Click **+**
3. Create interface assignments according to the table

| Lock | Description | Device |
| ---- | ----------- | ------ |
| Yes  | MANAGEMENT  | vlan01 |
| Yes  | TRUSTED     | vlan02 |
| Yes  | INTERNAL    | vlan03 |
| Yes  | DMZ         | vlan04 |
| Yes  | STORAGE     | vlan05 |

4. Click **Apply**

Lastly, we need to configure our VLAN network interfaces.

1. For each interface, navigate to Interfaces | `<interface-name>`
2. For each interface, configure according to the table:

| Name       | Enable | Lock | Description | Block private networks | Block bogon networks | IPv4 Configuration Type | IPv6 Configuration Type | MAC address | Promiscuous mode | MTU         | MSS         | Dynamic Gateway Policy | IPv4 Address | IPv4 Gateway Rules |
| ---------- | ------ | ---- | ----------- | ---------------------- | -------------------- | ----------------------- | ----------------------- | ----------- | ---------------- | ----------- | ----------- | ---------------------- | ------------ | ------------------ |
| MANAGEMENT | Yes    | Yes  | MANAGEMENT  | No                     | No                   | Static IPv4             | None                    | Leave blank | No               | Leave blank | Leave blank | No                     | 10.0.10.1/24 | Disabled           |
| TRUSTED    | Yes    | Yes  | TRUSTED     | No                     | No                   | Static IPv4             | None                    | Leave blank | No               | Leave blank | Leave blank | No                     | 10.0.20.1/24 | Disabled           |
| INTERNAL   | Yes    | Yes  | INTERNAL    | No                     | No                   | Static IPv4             | None                    | Leave blank | No               | Leave blank | Leave blank | No                     | 10.0.30.1/24 | Disabled           |
| DMZ        | Yes    | Yes  | DMZ         | No                     | No                   | Static IPv4             | None                    | Leave blank | No               | Leave blank | Leave blank | No                     | 10.0.40.1/24 | Disabled           |
| STORAGE    | Yes    | Yes  | STORAGE     | No                     | No                   | Static IPv4             | None                    | Leave blank | No               | Leave blank | Leave blank | No                     | 10.0.50.1/24 | Disabled           |

3. Click **Save** after each configuration
4. Click **Apply Changes**

# Closing Notes

You should now have an OPNsense firewall configured with the ability to deploy Proxmox virtual machines to their respective VLANs. In my next post I will discuss the deployment of a virtual machine for hosted services and the creation of firewall rules to manage and access them.
