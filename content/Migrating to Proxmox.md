---
publish: true
created: 2026-11-03T22:52:42.460Z
modified: 2026-11-12T00:43:00.656Z
tags:
  - homelab
  - rebuild
  - proxmox
  - ventoy
  - clonezilla
---

**A guide for installing, configuring, and migrating to Proxmox**
This is the first part of my homelab "rebuild" project, [[Homelab Infrastructure Rebuild Overview]]

# Migration

In order to migrate my homelab, I needed to back everything up. In order to do that, I had to actually figure out _where_ everything was, as I didn't really set anything up in a clean or consistent manner. I used this set of commands:

```
sudo docker ps --format '{{.Names}}\t{{.Image}}\t{{.Ports}}' > inventory.txt

sudo docker volume ls >> inventory.txt

sudo find / -name 'docker-compose.y*ml' -o -name 'compose.y*ml' 2>/dev/null >> inventory.txt
```

This creates a text file that logs an inventory of all pertinent docker information. The first command grabs a list of all running docker containers with their name, image, and published ports and adds it to the inventory file. The second adds all docker volumes to the inventory file. The third finds every compose file on the system and outputs the location of each compose file to the inventory file. Since I was exclusively using docker to host anything, this was all I needed. If you have other services, make sure to document those, including cronjobs.

After that, you'll need stop every service. You can use this command for docker:

```
docker stop $(docker ps -q)
```

The last thing to do is to backup your volumes and compose files to another storage solution, I used my NAS but a USB or any removable storage media works as well. Keep the inventory file as well.

It is also best practice to create a whole system iamge for backup, just in case. I recommend [Clonezilla](https://clonezilla.org/). See the section on [[#Ventoy and Clonezilla]] for more.

# Preparing for a Proxmox Install

Proxmox is a bare-metal hypervisor that will allow you to efficiently deploy multiple virtual machines on a single system. I replaced my singular Ubuntu instance with it. This opened up the opportunity to deploy different machines for different purposes in tandem with OPNsense for a firewall and segmentation, all on one machine.

To install Proxmox, you'll need the following things:

- Check that your motherboard's BIOS settings are correctly. You want to make sure IOMMU is enabled on your device. Different hardware sometimes has different names for this technology. Intel chipsets use VT-d, AMD chipsets use AMD-Vi, and ARM uses SMMU. It may be best to look up best settings for your specific hardware for Proxmox.
- An 8GB minimum USB drive.
- A reserved IP address for your Proxmox instance. You'll need to explore how to set this up for your home router.
- Create a bootable USB. For this I used [balenaEtcher](https://etcher.balena.io/). See the section on [[#Ventoy and Clonezilla]] for reasoning as to why I did not use Ventoy for Proxmox installation. To create a bootable USB with balenaEtcher:

1. Install [balenaEtcher](https://etcher.balena.io/) on a personal device.
2. Download the [latest Proxmox Virtual Environment installer ISO](https://www.proxmox.com/en/downloads/proxmox-virtual-environment). Make sure to grab the right one, if on ARM get the ARM ISO, otherwise get the standard ISO.
3. Plug your chosen USB drive into your device.
4. Open balenaEtcher
5. Click **Flash from file** and select your Proxmox installer ISO
6. Click **Select target**, tick the checkbox on `<your-USB-drive>`, click **Select**
7. Click **Flash**

Now you have a bootable USB drive to install Proxmox.

# Proxmox Installation and Basic Configuration

Before you do this, ensure that you have everything you need off your machine. **This will overwrite everything on a drive.**

1. Power off your future Proxmox machine.
2. Plug your bootable USB in to the machine.
3. Power your machine on.
   1. The following steps differ across hardware so you'll need to pay attention to your boot screen You should see a set of keys to press, press the one regarding your BIOS or UEFI.
   2. Boot your machine using your USB drive. Look up the how to do this for your specific hardware.
   3. Once you enter the Proxmox install screen, continue.
4. Select _Install Proxmox VE (Graphical)_ with the arrow keys, and press **Enter**
5. Click **I agree**
6. Select your Proxmox install disk in _Target Harddisk_
7. Click **Next**
8. Type `<your-country-of-residence>` into _Country_
9. Select your time zone in _Time zone_
10. Select the layout of your keyboard in _Keyboard Layout_
11. Click **Next**
12. Type `<your-secure-password>` into _Password_ and _Confirm_
13. Type `<your-email>` into _Email_
14. Click **Next**
15. Select your active network interface in _Management Interface_
16. Type `<your-hostname>` into _Hostname (FQDN)_. Pick something logical, like `pve.internal`.
    1. My future guides will discuss setting up internal DNS. If you have a domain scheme, you can apply it here, but it doesn't really matter since I'll be using Unbound DNS overrides for that.
17. Type `<your-reserved-ip>/24` into _IP Address (CIDR)_
18. Type `<your-gateway>` into _Gateway_
    1. Unless you have a custom setup, usually this is the first address of your subnet. So if you have a 10.0.0.X address, put 10.0.0.1
19. Type `<your-dns-server>` into _DNS Server_
    1. Similarly to the previous, unless you have a custom setup, your gateway's IP should work here.
20. Click **Next**
21. Click **Install**
22. Click **Reboot**

To configure Proxmox, you'll need to use a seperate device to access its management GUI. Proxmox automatically subscribes to its enterprise repository for updates, which requires a paid subscription. We want to make sure Proxmox is up to date and using the free repository. Here is how to do it:

1. Navigate to `<your-reserved-ip>:8006` in a web browser
2. Type `root` into _User name:_
3. Type `<your-secure-password>` into _Password:_
4. Click **Login**
   1. From here you should see a "No valid subscription" alert. This is because we are using free Proxmox, ignore it and click **OK**
5. Navigate to `<your-node>` | Updates | Repositories
   1. `<your-node>` should be your hostname or the first part of it. If you used `pve.internal` it should be `pve`
6. Looking at the _Components_ column, find and select the repository containing `enterprise`
7. Click **Disable**
   1. Repeat steps 6-7, looking for the repository containing `pve-enterprise` in the _Components_ column
8. Click **Add**
   1. You'll get another alert, click **OK**
9. Select _No-Subscription_ in _Repository:_
10. Click **Add**
11. Navigate to `<your-node>` | Updates
12. Click **Refresh**
13. Click the **X** icon when the task is done
14. Click **>\_ Upgrade**
15. Enter `Y` in the shell popup
16. Restart your server when the updates are complete. Click **Reboot | Yes**

Now we have a fully up-to-date instance of Proxmox with an update repository. To update your Proxmox, which you should do regularly, repeat steps 12-16.

# Ventoy and Clonezilla

[Ventoy](https://www.ventoy.net/en/index.html) is an amazing tool to create a bootable USB drive with multiple ISO files, rather than having a dedicated drive for each of your ISOs. I recommend creating a dedicated Ventoy USB.

I would advise against deploying Proxmox with Ventoy however, as I personally had errors with it, and the internet also seems to as well. Save yourself the headache, and don't use Ventoy to install it.

Ventoy is perfect for [Clonezilla](https://clonezilla.org/downloads.php), though since it is an ISO, and I would recommend using them together. Alternatively, you can flash Clonezilla to a drive with balenaEtcher, just as we did for Proxmox. To use Ventoy, you'll need to [download it](https://www.ventoy.net/en/download.html). Once you've unzipped the Ventoy file:

1. Open Ventoy2Disk.exe
2. Select `<your-dedicated-ventoy-usb>` in _Device_
3. Click **Install**

You should now have a USB device called Ventoy. Go ahead and move any ISOs you want on your Ventoy device to that device, including the [Clonezilla ISO](https://clonezilla.org/downloads.php).

To use Clonezilla, boot using your Ventoy device (Steps 1-3 of [[#Proxmox Installation and Basic Configuration]], using your Ventoy device instead). Make sure you have a storage medium prepared for your backup image, it can be a secondary drive, a removable storage medium, NAS, or another computer. Once you have booted with Ventoy:

1. Using arrow keys, select `<your-clonezilla-iso>`
2. Press **ENTER**
3. Select _Boot in normal mode_
4. Press **ENTER**
5. Select _Clonezilla live (VGA 800x600)_
6. Press **ENTER**
7. Select `<your-preferred-language>`
8. Press **ENTER**
9. Select `<your-keyboard-layout>`
10. Press **ENTER**
11. Select _Start\_Clonezilla_
12. Press **ENTER**
13. Select _device-image_
14. Press **ENTER**
15. Select `<your-preferred-storage-medium>`
    1. If you are using a NAS, choose _nfs\_server_ or _samba\_server_ for NFS or SMB respectively.
    2. If you are using a drive in your machine, or removeable media, choose _local\_dev_
    3. If you want to store it on another machine (which can be a NAS), use _ssh\_server_
16. Press **ENTER**
    1. At this point, steps may vary depending on your storage medium. Clonezilla will tell you what it needs based on your selection.
    2. If using USB to store your image, you'll want to insert it into the machine now. Press **CTRL+C** once it is recognized in the scan. Then select it from the partition list
    3. If using a network protocol (NFS, SMB, SSH) to store your image, Clonezilla will walk you through the necessary inputs to connect to these protocols
    4. If having trouble, refer to [these docs](https://clonezilla.org//fine-print-live-doc.php?path=clonezilla-live/doc/01_Save_disk_image)
    5. Start on step 17 once you get to a screen that says _Choose the mode to run the following wizard about advanced parameters_
17. Select _Beginner_
18. Press **ENTER**
19. Select _savedisk_
20. Press **ENTER**
21. Type in `<a-name-for-your-image>`
22. Press **ENTER**
23. Select `<your-disk-to-be-imaged>` (likely _sda_) and press **SPACE**
24. Press **ENTER**
25. Select _-z1p_
    1. Can use _-z9p_ if you have limited storage or a preference for it
26. Press **ENTER**
27. Select -_sfsck_
    1. This is usually fine, but if you believe the source filesystem is unhealthy, select _-fsck_ instead
28. Press **ENTER**
29. Select _Yes, check the saved image_
30. Press **ENTER**
31. Select _-senc_
    1. You may also select _-enc_ if you want to encrypt the image. Note that this will attach a passphrase to the image, and if forgotten, it will not be recoverable.
    2. You will be prompted for a password, if chosen, so do that before step 33.
32. Press **ENTER**
33. Select _-p choose_
34. Press **ENTER**
35. Press **ENTER**
36. Type `y`
37. Press **ENTER**
38. After Clonezilla completes, ensure you received a success message along the lines _All the images of partition or LV devcies in this image were checked and they are restorable:_ `<your-image-name>`
39. Press **ENTER**
40. Select _reboot_ or _poweroff_, depending on what you want to do
41. Press **ENTER**

You should now have a complete image of your machine on your chosen storage media device. To restore it, repeat steps 1-18, selecting your image directory on your storage media device. Then, instead of selecting _savedisk_, you'll want to use _restoredisk_. Clonezilla will then guide you through the restoration process. Let me know in comments if you would like to see a restoration guide.

# Closing Notes

I intentionally did not include VM deployment. This is because we want to deploy VMs behind our OPNsense firewall, which will also be a VM. See [[OPNsense Firewall Deployment, Configuration, and Segmentation with Proxmox]] for the deployment of an OPNsense firewall in your Proxmox environment.
