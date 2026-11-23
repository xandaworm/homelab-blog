---
publish: true
created: 2026-11-23T23:00:37.447Z
modified: 2026-11-23T23:13:58.418Z
tags:
  - homelab
  - clonezilla
  - ventoy
---

**A guide for the configuration of Ventoy and how to use Clonezilla**

I moved this topic from the [[Migrating to Proxmox]] guide, as it can serve as a standalone guide.

# Ventoy

[Ventoy](https://www.ventoy.net/en/index.html) is an amazing tool to create a bootable USB drive with multiple ISO files, rather than having a dedicated drive for each of your ISOs. I recommend creating a dedicated Ventoy USB.

Ventoy is perfect for [Clonezilla](https://clonezilla.org/downloads.php), though since it is an ISO, and I would recommend using them together. Alternatively, you can flash Clonezilla to a drive with balenaEtcher, just as we did for Proxmox. To use Ventoy, you'll need to [download it](https://www.ventoy.net/en/download.html). Once you've unzipped the Ventoy file:

1. Open Ventoy2Disk.exe
2. Select `<your-dedicated-ventoy-usb>` in _Device_
3. Click **Install**

You should now have a USB device called Ventoy(`<drive-letter>`:). Go ahead and move any ISOs you want on your Ventoy device to that drive, including the [Clonezilla ISO](https://clonezilla.org/downloads.php).

# Clonezilla

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

You should now have a complete image of your machine on your chosen storage media device. To restore it, repeat steps 1-18, selecting your image directory on your storage media device. Then, instead of selecting _savedisk_, you'll want to use _restoredisk_. Clonezilla will then guide you through the restoration process.

# Closing Notes

I'll likely periodically update this page with other bootable tools.
