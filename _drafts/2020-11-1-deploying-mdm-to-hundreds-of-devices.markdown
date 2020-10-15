---
title: How we deployed MDM (Workspace One) to hundreds of devices
excerpt: This blog post summarizes the thoughts and processes behind our deployment of 
link: https://andrewdoering.org/blog/2020/11/01/deploying-mdm-to-hundreds-of-devices
slug: deploying-mdm-to-hundreds-of-devices
author: Andrew Doering
 #Remove this after finishing the document
comments: true
date: 2020-11-1 01:00:00 -0700
last_modified_at: 
layout: post
tags:
- installapplications
- nudge
- macOS
- workspace_one
- zero touch provisioning
- windows
- okta
- DEP
- MDM
- device enrollment program
- mobile device management
categories:
- mdm
- provisioning
- macOS
- Windows
img_bg: /assets/blog/2020/10/
img_bg_alt: Placeholder Alt text
img_blog: /assets/blog/2020/10/ # Image must be 800 x 650
img_alt: Placeholder Alt Text
permalink: /blog/:year/:month/:day/:title/
---

- [Introduction](#introduction)
  - [Initial Deployment Phases](#initial-deployment-phases)
    - [What we used on macOS:](#what-we-used-on-macos)
    - [What we used on Windows:](#what-we-used-on-windows)
  - [MDM Deployment Phase](#mdm-deployment-phase)
    - [What we used on macOS:](#what-we-used-on-macos-1)
    - [Workspace One Configuration](#workspace-one-configuration)
      - [Split paths for existing and new devices](#split-paths-for-existing-and-new-devices)
        - [Existing devices](#existing-devices)
        - [New / Zero Touch Provisioning System](#new--zero-touch-provisioning-system)
  - [Moving forward](#moving-forward)

## Introduction

At [$previouscompany](https://andrewdoering.org/#resume) we had gone through a long struggle of managing devices properly. When I was hired in 2016, it didn't help that I was hired having never actively used a macOS device before in my life - and admittedly I was out of scope, but doubled down and did a lot of research off and on hours. Our initial deployment of a provisioning system was using DeployStudio. It worked (slowly and barely) for two years (from 2016 - 2018), and admittedly, we were still a small company at this point, roughly 250 max, however, the mistake had already been made as our biggest two years of growth were during this time period. However to continue to scale the companies operations, we needed to get an efficient and flexible system.

After an uphill battle, countless research documents, proposals, and explanations, we finally reached a point where it was agreed on that we would obtain an MDM solution. Going to detail exactly what we went through in terms of deployments before MDM, during/after MDM, and going forward (with Windows 10 and Big Sur).


### Initial Deployment Phases 

This is for the time period between 2016 and the end of 2017.

#### What we used on macOS:

* [DeployStudio](https://www.deploystudio.com/)
* [AutoDMG](https://github.com/MagerValp/AutoDMG)
* [Munki](https://www.munki.org/)
* [Sal](https://github.com/salopensource)
* [Crypt Client](https://github.com/grahamgilbert/crypt)
* [Crypt Server](https://github.com/grahamgilbert/Crypt-Server)


To go into detail on this, we would create a prepared image from AutoDMG with the latest updates. We would go the "Golden Image" route. Once the golden image was prepared (which was a thin image, not a monolithic image), we would then drop it on our DeployStudio system and began the deployment process.

We had several different workflows for each department and team, pre-installing the necessary software for them ahead of their onboarding (or at least what IT was told about). These would be installed after the image was installed on each machine. 

There were a few issues here that came out of this, the largest one was the ability to support macOS images that would become branched due to hardware revisions. The other was once the T2 chips were released, we couldn't really use DeployStudio anymore.

#### What we used on Windows:

* OEM Windows 10 Image
* [Crypt Client](https://github.com/johnnyramos/bitlocker2crypt)
* Group Policy Scripts

For our Windows deployment, there was minimal work needed, as we scripted up most of our onboarding processes, we could have defnitely 

### MDM Deployment Phase

#### What we used on macOS:

* [Munki](https://www.munki.org/)
* [Sal](https://github.com/salopensource)
* [Crypt Client](https://github.com/grahamgilbert/crypt)
* [Crypt Server](https://github.com/grahamgilbert/Crypt-Server)
* [Privileges](https://github.com/SAP/macOS-enterprise-privileges)
* [umad](https://github.com/macadmins/umad)
* [nudge](https://github.com/macadmins/nudge) (and a more [feature rich fork exists](https://github.com/LcTrKiD/nudge))
* [installapplications](https://github.com/macadmins/installapplications) 

Since we previously had already rolled out Munki to our fleet, this made the MDM enrollment a lot easier.


#### Workspace One Configuration

I have already written up on our [SAML and Directory Services deployment of Workspace One](https://andrewdoering.org/blog/2019/12/23/workspace-one-okta-users-and-groups-working-with-dep-enrollment/), however I have not described our existing setup. 

##### Split paths for existing and new devices

The requirements we had setup were:

1. Enroll existing devices
2. Enroll all new devices via Zero Touch Provisioning Scheme
3. Do nothing for Bring your own Device (ByoD)
4. Do nothing for Employee Owned / Personal Devices (EOPD)

While ultimately, internally we did want to create programs for the bottom two items, we were not able to get anything greenlighted internally. Within IT, we saw the bottom two as potential risks, and the long term effort was to enable device trust and context based access via Okta to minimize data loss and that risk. However there were issues at play from above that did not allow us to execute 3 or 4.

###### Existing devices

We created an organizational group specifically for previous devices within the company. 


###### New / Zero Touch Provisioning System

Again, we created an organizational group specifically for new devices. This was meant to be a way that we could easily monitor devices being registered on a weekly basis for new employees. We had several other methods of doing this (eg: Sal), but this way, our IT team, InfoSec teams, and other teams could pull data out of Workspace One for all new devices rather than have the entire list of all previous devices.


### Moving forward

Now that we are being acquired, all of our device management plan has been scrapped.However, with that said, I do want to make some comments on the direction of Big Sur and how the zero-touch provisioning system is changing.

A lot of installapplications use cases come from the userland configuration to display context to a user once it hits the login screen. However, there have been [reports](https://macadmins.slack.com/archives/C54N3AT2B/p1601644844008600) of this [failing](https://macadmins.slack.com/archives/C54N3AT2B/p1601408174001500) or not working as expected with ``11.0b*`.