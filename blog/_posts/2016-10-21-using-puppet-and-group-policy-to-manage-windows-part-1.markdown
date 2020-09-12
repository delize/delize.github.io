---
author: Andrew Doering
comments: true
date: 2016-10-21 23:00:53+00:00
layout: post
link: https://andrewdoering.org/blog/2016/10/21/using-puppet-and-group-policy-to-manage-windows-part-1/
published: false
slug: using-puppet-and-group-policy-to-manage-windows-part-1
title: Using Puppet and Group Policy to manage Windows (part 1)
wordpress_id: 71
tags:
- Active Directory
- Powershell
- Puppet
- Server
- Windows
---

Automation of technology is becoming a rapidly apparent need. No longer can setup be a manual process of creating an IIS or Active Directory server using Role Manager. It has to be done quickly and autonomously. After searching for several good guides that explained in depth how I could use Puppet to assist with managing a server that would be destined for Active Directory (we really don’t use IIS within the company), I found zero articles that could adequately explain in depth with code sample, photos, etc. So with my best efforts, I’ll attempt to do that here. This is my first attempt to use Puppet. The initial reason for all of this is:




    
  * There is no Change Management with Group Policy Manager

    
  * There is no Desired Configuration State with Group Policy Manager

    
  * Reporting features of Group Policy don't function incredibly well



For an example, I have a group policy object that is intended to disable the screen saver. It worked well for roughly a month, and now it no longer works – no change was made to the GPO. Even with Resultant Set of Policy planning stating that the GPO is in effect, it avoids. Another example, software installations attempting to be pushed – which just decide to not install even with proper configurations. All of these policies are set in Enforced Mode.

Group Policy is hard to manage efficiently and correctly, or in other words, it is incredibly difficult to troubleshoot. Even if a policy is applied, there could some other policy overriding those changes. It’s a devastating cycle. There is change management tool for Group Policy Objects from Microsoft called [Advanced Group Policy Manager](https://technet.microsoft.com/en-us/windows/hh826067.aspx) and it is available through [Microsoft Desktop Optimization Pack](https://msdn.microsoft.com/en-us/subscriptions/downloads/#FileId=65215). Changes can be made outside of this tool, and then the next time someone "checks out" a group policy, edits it, and then checks it back in, the anonymous changes are also committed. So... I mean, that is so helpful. Microsoft has [released an article](https://blogs.technet.microsoft.com/askds/2011/06/21/forcing-domain-admins-to-use-agpm-but-not-really/) about this, but the problem with Domain Admins is, they can do EVERYTHING! That means these tools are at the end of the day, not the right change management tool I am looking for.

So with that said, the idea that I have is to use a combination of Group Policy, and Puppet for the initial configuration. Eventually, I would like to use System Configuration Change Management (SCCM), however, doing one thing at a time, lets delve into Puppet first.



#### So what are the primary goals?



To create a baseline to be applied to all Windows servers, prior to their enrollment to either an Active Directory enrollment or prior to the servers joining the domain.



#### What would be useful in this situation?





##### Puppet



This software works well for machines that are not attached to a domain, and need an initial automatic configuration prior to being attached to the domain. Everything is managed easily via a (or a group of) text files, and that suits my needs exactly. While the grouping of the files (through Heira, yaml files, .pp files, modules, etc) is initially a little difficult to understand, you will get the hang of it. One of my recommendations, don't start with module configurations. This will only make it so much more challenging, as you will be starting from the bottom and working your way to the top.



##### Group Policy Objects



Group Policy can work effectively, when done properly and is an easy top level way to manage settings without installing “agents” on laptops. But as previously mentioned, it is a right pain to use. With that being said we will use Group Policy Object as a fall back for everything if required. Along with that, certain things cannot be configured via Puppet and the mixture of modules that can be used. So with the specifics that Puppet cannot administer, we will use Group Policy Objects.



#### What are the constraints to this?



The largest issues, are the overlapping objects. If you have one object/resource configured in puppet and change it in GPM (or vice-a-versa), you have a problem. Obviously the two groups will start conflicting and it will be hard to manage or determine what is needed to happen. Imaging a server installation is not a fun experience. There are still steps involved to get to the automation process initially running.

Next part will be digging into the actual configuration files, and how I am going to manage the Windows Puppet configuration for our servers.
