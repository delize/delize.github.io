---
title: How we deployed MDM (Workspace One) to hundreds of devices
excerpt: This blog post summarizes the thoughts and processes behind our deployment of workspace one to several hundred devices on both Windows and macOS.
link: https://andrewdoering.org/blog/2020/10/21/deploying-mdm-to-hundreds-of-devices
#slug: deploying-mdm-to-hundreds-of-devices
author: Andrew Doering
 #Remove this after finishing the document
comments: true
date: 2020-10-21 01:00:00 -0700
last_modified_at: 
layout: post
tags:
- installapplications
- nudge
- macOS
- workspace_one
- zero touch provisioning
- windows
- Okta
- DEP
- MDM
- device enrollment program
- mobile device management
- zero-touch provisioning
categories:
- Mobile Device Management
- macOS
- Windows
img_bg: /assets/blog/2020/10/mdm/background.webp
img_bg_alt: Workspace One MDM Deployment to hundreds of devices.
img_blog: /assets/blog/2020/10/mdm/smaller_icon.jpg # Image must be 800 x 650
img_alt: Blog micro image
permalink: /blog/:year/:month/:day/:title/
---

- [Initial Deployment Process](#initial-deployment-process)
  - [What we used on macOS](#what-we-used-on-macos)
  - [What we used on Windows](#what-we-used-on-windows)
- [MDM Deployment Phase](#mdm-deployment-phase)
  - [Prerequisites](#prerequisites)
    - [What we used on macOS](#what-we-used-on-macos-1)
    - [What we used on Windows](#what-we-used-on-windows-1)
    - [Registering devices from our Vendor with Apple Business Manager](#registering-devices-from-our-vendor-with-apple-business-manager)
      - [Apple ECommerce Websites](#apple-ecommerce-websites)
      - [CDW](#cdw)
      - [SHI](#shi)
  - [Workspace One Configuration & Workflow](#workspace-one-configuration--workflow)
    - [Scope of devices for enrollment/management](#scope-of-devices-for-enrollmentmanagement)
    - [Existing devices](#existing-devices)
      - [macOS](#macos)
      - [Windows](#windows)
    - [Zero Touch Provisioning System](#zero-touch-provisioning-system)
      - [macOS](#macos-1)
        - [installapplications](#installapplications)
        - [How we deployed Munki](#how-we-deployed-munki)
        - [How new employees gain admin access](#how-new-employees-gain-admin-access)
        - [User Experience](#user-experience)
      - [Windows](#windows-1)
        - [Autopilot / OOBE](#autopilot--oobe)
        - [Deployment Process ( ~2 Years Later)](#deployment-process--2-years-later)
        - [Chef and Chocolatey](#chef-and-chocolatey)
  - [Summary](#summary)
- [Moving forward](#moving-forward)
  - [Outside of $currentcompany](#outside-of-currentcompany)
  - [Within $currentcompany](#within-currentcompany)
- [Supporting Documents](#supporting-documents)

At [$previouscompany](https://andrewdoering.org/#resume) we had gone through a long struggle of managing devices properly. When I was hired in 2016, we had nothing in place for our entire company and everything was done manually. Our initial provisioning system focused on macOS as that was the most widely used OS within the company. It worked (slowly and barely) for two years (from 2016 - ~2018), and admittedly, we were still a small company at this point, roughly 250 max, however, the mistake had already been made as our biggest two years of growth were during this time period. However to continue to scale the companies operations, we needed to get an efficient and flexible system that incorporated both macOS and Windows.

After an uphill battle, countless research documents, proposals, and explanations, we finally reached a point where it was agreed on that we would obtain an MDM solution. This will detail exactly what we went through in terms of deployments before MDM, during/after MDM, and going forward (with Windows 10 and Big Sur).

This blog post has been written post-acquisition and about a year after our deployment of this process, this is not a summary of what have in active use, but the fundamentals we have used in the past. I attempted to make this with as less ranting and going on a tangent as possible. We will no longer be using this deployment method due to the acquisition.

## Initial Deployment Process

This is for the time period between 2016 and the end of 2017.

### What we used on macOS

- [DeployStudio](https://www.deploystudio.com/)
- [AutoDMG](https://github.com/MagerValp/AutoDMG)
- [Munki](https://www.munki.org/)
- [Sal](https://github.com/salopensource)
- [Crypt Client](https://github.com/grahamgilbert/crypt)
- [Crypt Server](https://github.com/grahamgilbert/Crypt-Server)
- [Local LAPSmac](https://github.com/delize/local-LAPS-python)

To go into detail on this, we would create a prepared image from AutoDMG with the latest updates. We would go the "Golden Image" route. Once the golden image was prepared (which was a thin image, not a monolithic image), we would then drop it on our DeployStudio system and began the deployment process.

We had several different workflows for each department and team, pre-installing the necessary software for them ahead of their onboarding (or at least what IT was told about). These would be installed after the image was installed on each machine.

There were a few issues here that came out of this, the largest one was the ability to support macOS images that would become branched due to hardware revisions. The other was once the T2 chips were released, we couldn't really use DeployStudio anymore, and finally manual builds were just overall time consuming and not worth it in the long run. No one should be manually building laptops at this point of device lifecycles (looking at you $currentcompany).

### What we used on Windows

- OEM Windows 10 Image
- [Crypt Client](https://github.com/johnnyramos/bitlocker2crypt)
- Group Policy Scripts

For our Windows deployment, there was minimal work needed, as we scripted up most of our onboarding processes, we definitely had some improvements to be made here, however Windows was always considered a lower priority compared to macOS - even tough about ~25% of our workforce was Windows based clients, and we have several build machines, and servers that were Windows based.

We were also told that trying to get statistical and reporting on our Windows environment, to assist with evidence and provide data that we needed to improve our Windows environment, was not currently relevant when we went through this project (much to my disagreement, we abided by this and didn't capture details). Even historical data to provide future projections were ignored. My reaction to these conversations is summed up by the following meme:

![Ryan Reynolds](/assets/blog/2020/10/mdm/giphy.gif)

## MDM Deployment Phase

This project was initiated at the beginning of 2018 and completed by mid-2018, the fundamental pieces of this workflow was used during and up to the acquisition.

### Prerequisites

#### What we used on macOS

- [Munki](https://www.munki.org/)
- [Sal](https://github.com/salopensource)
- [Crypt Client](https://github.com/grahamgilbert/crypt)
- [Crypt Server](https://github.com/grahamgilbert/Crypt-Server)
- [Privileges](https://github.com/SAP/macOS-enterprise-privileges)
- [umad](https://github.com/macadmins/umad)
- [nudge](https://github.com/macadmins/nudge) (and a more [feature rich fork exists](https://github.com/LcTrKiD/nudge))
- [installapplications](https://github.com/macadmins/installapplications)
- [DEPNotify](https://gitlab.com/Mactroll/DEPNotify)
- [outset](https://github.com/chilcote/outset)
- [Developer ID Installer Certificate](https://developer.apple.com/support/certificates/)
- [Apple Business Manager](https://business.apple.com/)

Since we previously using Munki throughout our fleet, this made the MDM enrollment a lot easier.

#### What we used on Windows

- [Windows Autopilot](https://www.microsoft.com/en-us/microsoft-365/windows/windows-autopilot) / [OOBE](https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/1903/Windows_Desktop_Device_Management/GUID-AWT-ENROLL-OOBE.html)
- [Chef](https://www.chef.io/)
- [Chocolatey](https://chocolatey.org/products/chocolatey-for-business)
- [Crypt Client](https://github.com/johnnyramos/bitlocker2crypt)

#### Registering devices from our Vendor with Apple Business Manager

We generally purchased all devices through CDW, SHI, or directly through an Apple Business Account based on location. For Apple, each reseller company has a DEP Reseller ID, and adding that DEP Reseller ID to your Business Account allows them to automatically register devices back into your MDM workflow.

##### Apple ECommerce Websites

We had apple business accounts in all of the regions that provide a business platform, for our business location registrations. [This](https://ecommerce.apple.com/content/b2b/static/en/us/flags.html) was very helpful, however there were limitations. The main limitation was a lack of automation around pulling assets into an asset management system.

##### CDW

We utilized a script that I wrote [here](https://github.com/delize/snipeit_cdw_automation) to upload data into SnipeIT. From there, we allowed CDW access to our environment for Apple Business Manager to upload device data. After initially looking at [Autopilot](https://www.microsoft.com/en-us/microsoft-365/windows/windows-autopilot), [due to cost](https://www.reddit.com/r/sysadmin/comments/8hpqoe/anybody_using_windows_10_autopilot/) however, the same reasons in this reddit thread, we opted to go a different route. This is no fault to CDW, but the cost of Autopilot is absurd when similar services in other vendors are free and Microsoft offers it for free on Surface devices. If we were to deploy 1000 machines, at the cost of $10.00 per machine, $10,000.00 in addition to the laptop amount purchased.

##### SHI

Generally we don't purchase many EUC devices through SHI, but we added the DEP Reseller IDs to our ABM account anyways. We did not setup an integration for AzureAD/Windows Devices.

### Workspace One Configuration & Workflow

I have already written up on our [SAML and Directory Services deployment of Workspace One](https://andrewdoering.org/blog/2019/12/23/workspace-one-okta-users-and-groups-working-with-dep-enrollment/), however I have not described our existing setup.

#### Scope of devices for enrollment/management

The requirements we wanted to roll out:

1. Enroll existing devices
2. Enroll all new devices via Zero Touch Provisioning Scheme
3. Allow users to enroll their mobile phone(s), tablets, and personal devices

The requirements we ended up with were:

1. Enroll existing devices
2. Enroll all new devices via Zero Touch Provisioning Scheme
3. Do nothing for mobile phones (BYOD)
4. Do nothing for Employee Owned / Personal Computing Devices (EOPD / BYOD)

Internally we did want to create programs for the bottom two items, we were not able to get anything green lit internally. Within IT, we saw the bottom two as potential risks, and the long term effort was to enable device trust and context based access via Okta to minimize data loss and that risk. We do not distribute phones from a company owned mobile plan. However there were issues at play from above that did not allow us to execute 3 or 4.

With the scope settled, we took two different paths for existing vs new devices.

#### Existing devices

##### macOS

We created an organizational group specifically within Workspace One for previous devices within the company. Here we utilized [umad](https://github.com/macadmins/umad) a tool written by [Erik Gomez](https://blog.eriknicolasgomez.com/) and had two routes that would be taken:

- umad + DEP
- umad + manual

The bulk of our devices were DEP capable directly after deploying umad, and the majority of our team enrolled smoothly.

From there, our first line support team (who were also our IT engineering team, yay for teams of 2!) would follow up on devices after about a week or two of deployment time. This was enough time for employees to enroll themselves, and in addition for the timer to kick in with umad to the point where it would become annoying enough for employees that did not enroll (to require an incident to be opened to either stop the pop ups or enroll into MDM).

##### Windows

Windows has a self registration portal for enrolling devices into an MDM, located under the "Access work or school" in system settings, on the right side pane labeled `Enroll only in Device Management`. Utilizing this, users could self-enroll their devices as long as they had administrator access and secondary/additional software was not needed like it was for macOS.

#### Zero Touch Provisioning System

Again, we created an organizational group specifically for new devices. This was meant to be a way that we could easily monitor devices being registered on a weekly basis for new employees. We had several other methods of doing this (eg: Sal), but this way, our IT team, InfoSec teams, and other teams could pull data out of Workspace One for all new devices rather than have the entire list of all previous devices.

##### macOS

###### installapplications

We started development of this workflow with installapplications with version 1.0 and ended up finalizing the original deployment using 1.2.1. There is a [great example repository here](https://github.com/erikng/installapplicationsdemo). We used this as our base example, but removed a substantial amount of scripts that were not relevant to us.

###### How we deployed Munki

We compiled a smaller/custom version of Munki that allows us to do a single run of Munki against our web server hosting our manifests. With this single run, we implement a bootstrap manifest that runs once, and then our custom auto enroll kicks in and allows us to use the serial number subsequent Munki runs.

Rolling our own:

There is some documentation and several blog posts on how to compile Munki, rather than going in depth on how we did this and due to the length since we did this, I would prefer to [link](https://blog.eriknicolasgomez.com/2017/03/08/Custom-DEP-Part-2-Creating-a-custom-package-and-deploying-Munki/) [different](https://tombridge.com/2017/04/27/getting-started-with-installapplication-depnotify-and-simplemdm/) [posts](https://therestoftheowl.com/blog/2018/11/09/installapplications-for-dummies-part-1/) here.

Manifests:

We use a similar Munki deployment that follows the same principles as [Munki Enroll](https://github.com/edingc/munki-enroll) and [Munki Serial Enroll](https://github.com/aysiu/munki-serial-enroll). However, our tool was internally developed and has not been released - namely our service does not rely on PHP. Our structure of our repos is described below:

![Structure Diagram](/assets/blog/2020/10/mdm/structure.png)

This way, we had the flexibility to deploy to a device, to a team/department, to a region, and globally across the domain.

###### How new employees gain admin access

Before our acquisition, we did not grant admin access across the board to the company (due to InfoSec requirements), we only allowed a few departments (namely: engineering, InfoSec, customer success, and pre-sales solution engineers). Due to the flexibility of our Munki enrollments, it was quite easy to accomplish in this way.

[Rich Trouton](https://derflounder.wordpress.com/) at SAP has published [Privileges.app](https://github.com/SAP/macOS-enterprise-privileges/). We use this in certain Munki manifests to automatically install and grant access, while also setting up a post install script to automatically run and enable admin access after Munki installed the application. We could do this natively [with Munki](https://github.com/munki/munki/wiki/Managing-Admin-Rights-With-Munki), however we wanted to provide the opportunity for our engineers to dynamically switch admin context on the fly, and SAP's tool allows us to do this.

###### User Experience

We heavily utilized [DEPNotify](https://gitlab.com/Mactroll/DEPNotify) to inform our employees of the current status of their new device. A very generic image of our initial deployment (which can be seen by many other blogs) can be found below, however, we customized this a bit with stories on how ThousandEyes operates, our culture, and our history.

![DEPNotify experience](/assets/blog/2020/10/mdm/macOS%20New%20Hire%20Guide_Page_6_Image_0002.png)

After this, we would then reset the dock in place to set our generally most used services. This would consist of actual applications, or links/shortcuts to web applications (mainly G Suite services).

We wanted to minimize the deployment time, configuration of a device shouldn't take more than 15 to 20 minutes, any longer (in our mind, 30 minutes or more) and not only does it ruin the experience, it also is detrimental to the new employee as more could go wrong with the deployment. To make sure our intended deployment was less than 20 minutes we simulated a few environments. A 1 Megabit connection (worst case scenario), 1 Megabyte, 5 Megabytes, and Full Speed (in my testing - that is a 1 Gigabyte downstream on a home ISP). In testing the worst case scenario ended up being 23 minutes where the best case scenario ended up being between 10 - 15 minutes in a virtualized ESXi machine. We considered this successful.

If something does go wrong with the deployment, we also created a package that would fire off a stand alone setup package that would install all the necessary tools that would have been installed during the first experience.

Once their initial installation was finished, we would run JAMF Connect Sync on reboot logins to prompt the user to sync their password down from Okta to their local device.

![JAMF Connect Sync](/assets/blog/2020/10/mdm/macOS%20New%20Hire%20Guide_Page_8_Image_0001.png)

##### Windows

The forgotten step-child, as I mentioned, trying to get anything done within our environment on Windows was difficult as management deemed it unnecessary. All the while our fleet deployment of windows was ~25% of our total systems. Fortunately, the improvements we made with macOS (and MDM in general) also were baked into (to a minimal degree) for Windows as well. Meaning, that remote employees and workforces could be rolled out without someone sitting next to the computer for 30 to 60 minutes configuring a PC. Unfortunately, we couldn't use it in the provisioning process due to upper management.

However, once covid happened, we had sudden approval to initiate Windows improvements.

###### Autopilot / OOBE

Workspace One has a nice guide on how to setup both AutoPilot and OOBE that can be [found here](https://techzone.vmware.com/enrolling-windows-10-devices-using-azure-ad-vmware-workspace-one-uem-operational-tutorial#1108817). This was the base we used to get our configuration off the ground and running. We did run into a hitch that I will describe a little later on.

###### Deployment Process ( ~2 Years Later)

For the Windows side, we configured [auto-discovery](https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/services/System_Settings_On_Prem/GUID-AWT-DEVICESUSERS-WINDOWS-WADS.html) feature that Workspace One supports. This (minimally) shortens manual process by 2 to 3 steps. For automatic steps, it generally can help automate the entire process where end users do not have to fill in server configurations.

![Workspace One Auto Discovery](/assets/blog/2020/10/mdm/Screen%20Shot%202020-10-17%20at%207.49.48%20PM.png)

While rolling this out for zero-touch, we did run into a single issue which unfortunately was a show stopper for zero-touch only. That is, when you have Active Directory previously hooked up to Okta, it causes issues when mapping `uniqueidentifiers` vs `ObjectGUID`. We ultimately had to open up a support case to attempt to resolve this, and then were redirected to ProServices to fix the problem. By the time we got approval for ProServices, the announcement of an acquisition was announced the next day. The annoying part about this issue, is that it ONLY presented itself when utilizing AzureAD's Workspace One VMware app during OOBE experience. If manual sign up using AzureAD Domain Join and manual enrollment into Workspace One MDM, it was not present.

![External ID Error](/assets/blog/2020/10/mdm/Screen%20Shot%202020-10-17%20at%208.27.16%20PM.png)

However, because we already have 400+ accounts in O365, and an immutableID is well, immutable, we were not sure how to go through with this without causing massive issues. Unfortunately, this was never brought up to us until far after, and while writing this, I realized that there was some misconfigurations when going through our Professional Services with Workspace One that did not take into account this configuration in Okta and AzureAD.

While Okta being federated does imitate ADSync, our federated configuration does not actually show up as an External Identity in AzureAD's configuration.

It does seem that Microsoft has finally allowed the ability to [create custom attributes in AzureAD](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/user-flow-add-custom-attributes). This could have helped us with a workaround. However, if anyone **does** know of a way to fix this issue, I would honestly love to hear it. There is [a lot](https://support.okta.com/help/servlet/fileField?id=0BE2A00000004Sj) of [documentation](https://www.okta.com/sites/default/files/Okta-3rd-Party-UEM-Interop_Workspace-ONE.pdf) [I have](https://help.okta.com/en/prod/Content/Topics/Apps/Office365-Deployment/provision-users.htm) read in the past, however even if we can't/don't use it going forward, just for completion and a way to close a loop here.

We unfortunately went the manual route. Requiring users to sign in manually to Azure AD, and then manually sign into the MDM.

###### Chef and Chocolatey

Once this was done, we setup the remaining equivalent scripts, and deployed our required applications for a short term roll out.

Long term, the plan was to deploy [Chef on Windows](https://docs.chef.io/windows/). With that, we would deploy [chocolatey](tey.org/products/chocolatey-for-business) and allow for a self service deployment of software similar to SCCM, but with far less expense.

While we had a high level plan on the end states, we did not have a low level plan, and one of my peers wanted to take that over to get a better understanding of the NuGet framework. The idea here was that we would have Chef managing devices similar to Munki but utilizing the cookbooks of other companies like Facebook, Uber, and others. Once we got a baseline we would then start developing our own to utilize it more to our needs.

### Summary

As a reminder, these photos were taken during our initial deployments (and our documentation) and are not a current status of what we do, a lot of continual improvements have been performed throughout our initial deployment, and revamps have been made to customize this to exactly the way we want our users to experience deployments.

Minimal to no input, while still having a great deal of flexibility in the deployment process. People should not have to question where to find details like Asset Tags, Serial Numbers, etc. However they should have the ability to fully customize their device.

With all this setup and configured, it worked well and allowed a lot more time for our IT team that focused on engineering to take up new challenges, and allow our support team to have more time to focus on other services and functions that they need to worry about.

## Moving forward

### Outside of $currentcompany

Now that we are being acquired, all of our device management plan has been scrapped. However, with that said, I do want to make some comments on the direction of Big Sur and how the zero-touch provisioning system is changing.

First and foremost, Python is getting removed in Big Sur. Meaning that you have to have roll your own in the deployment process. With `installapplications`, this is no longer a problem as there is a version released with compatible python, however, this leads to other potential issues. The `mdmclient` on macOS is rather finicky and not entirely reliable. Since we now have to bundle python and `installapplications`, it raises the size from ~25 kilobytes to ~25 megabytes.

A huge component of `installapplications` use cases come from the ability to display and load user land context for the user once they hit the desktop environment. However, there have been [reports](https://macadmins.slack.com/archives/C54N3AT2B/p1601644844008600) of this [failing](https://macadmins.slack.com/archives/C54N3AT2B/p1601408174001500) or not working as expected with macOS 11.0 Beta / Big Sur. Granted that due to Big Sur being in beta, there may be some unexpected behavior, but it seems the best route going forward is to no longer use DEP Notify directly launched from `installapplications`. Rather, use outset on first boot and set a flag to signify if DEPNotify had been run, and if not run it.

Due to our deprecation of services, I am interested to see what comes out of Big Sur and Windows 10, and have been closely following both the [Windows discord](https://discord.me/winadmins) as well as the [MacAdmins Slack](https://www.macadmins.org/).

### Within $currentcompany

It will be interesting to see how a large legacy centric company attempts to convert to a cloud centric company with "traditional management" frameworks. The lack of use of Chef or Puppet on EUC devices, while not a limiting factor, does limit a lot of imagination and expansion room.

## Supporting Documents

While I don't feel comfortable publishing our documentations around our processes, happy to distribute them privately if anyone may potentially need them or want to look at them. Feel free to comment below, and/or send me an [email](mailto:contact@andrewdoering.org).
