---
author: Andrew Doering
comments: true
date: 2018-03-27 23:56:31 PT
layout: post
link: https://andrewdoering.org/blog/2018/03/27/vmware-esxi-macos-10-13-apfs/
excerpt: This is a verbatim copy of a blog post written by "licson" as his site was going down.
slug: vmware-esxi-macos-10-13-apfs
title: vmware / ESXi + macOS 10.13 APFS
wordpress_id: 452
comments: true
permalink: /blog/:year/:month/:day/:title/
---

First off, this is a verbatim copy of a blog post written [here](https://licson.net/post/vmware-apfs/) ([https://licson.net/post/vmware-apfs/](https://licson.net/post/vmware-apfs/)) and a cached content version [here](https://webcache.googleusercontent.com/search?q=cache:Uvv6UFhC2GsJ:https://licson.net/post/vmware-apfs/+&cd=1&hl=en&ct=clnk&gl=us)

I don't take credit for the content of this at all, the developer deserves all credit for his ingenuity, but I am reposting this as it is incredibly useful, and the current blog has gone off-line due to a database issue.



# Using Apple File System (APFS) with your virtualized Mac



Apple has just released the macOS High Sierra with new features, one of them is the brand-new Apple File System (APFS) that is optimized for flash storage which newer Macs enjoy. If you happen to be using macOS in a virtualized way, e.g. with VMware, you may have trouble getting the new OS to work as the upgrade forces conversion of the boot partition to APFS which the VMware UEFI does not support.

To solve the problem, we need to let the VMware UEFI know APFS and luckily the APFS driver can be extracted from the High Sierra installer as a UEFI driver executable. We can then slip the driver to the UEFI BIOS that bundles with VMware Player itself and everything should work.



## Getting Started



We’ll need 3 things before modifying the VMware UEFI BIOS. They are listed below:





  * [The APFS UEFI Driver extract](https://github.com/darkhandz/XPS15-9550-Sierra/blob/master/CLOVER-Install/drivers64UEFI/apfs.efi?raw=true)


  * [UEFITool, a tool for editing UEFI BIOS](https://github.com/LongSoft/UEFITool/releases)


  * [FFS to convert the APFS driver to UEFI module](https://github.com/pbatard/ffs/releases)



To simplify things, you can download my [efi64_apfs.rom](/assets/img/2018-03-27-post/efi64_apfs.rom_.zip) (tested on VMware Workstation Pro 14, may work for other versions too). If that ROM doesn’t work for you, go after these steps to get a modified BIOS with APFS support.

Use UEFITool to open EFI64.rom located at [VMware Installation Folder]/x64/, select File > Search and choose GUID tab. Type in 961578FE-B6B7-44C3-AF35-6BC705CD2B1F and double click the result inside Message section. Leave this screen for now.

[![UEFITool](/assets/img/2018-03-27-post/vmware_apfs_ubu_screen.png)]

Extract the FFS tool to the same directory as the APFS driver file. Open your command prompt, change directory to that place and run this command:  GenMod apfs.efi .

[![](/assets/img/2018-03-27-post/vmware_apfs_ffs_screen.png)]

Go back to UEFITool, right-click the selected item and choose Insert After, then select apfs.ffs from the FFS directory. The screen should look like this.

[![VMware APFS INS Screen](/assets/img/2018-03-27-post/vmware_apfs_ins_screen.png)]

Save the modified ROM with the name efi64_apfs.rom to your VM directory.



## Applying the new UEFI BIOS



To get the modified UEFI BIOS to work, use a text editor to open the VMX file. Ensure the file contains the following lines.


    
    <code>firmware = "efi"
    efi64.filename = "efi64_apfs.rom"
    </code>



Save the VMX file and start your VM, your macOS High Sierra will now boot as expected with an APFS volume. Voila!
