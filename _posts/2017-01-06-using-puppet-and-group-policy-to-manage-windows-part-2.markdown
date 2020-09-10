---
author: andrewdoering
comments: true
date: 2017-01-06 05:31:24+00:00
layout: post
link: https://andrewdoering.org/blog/2017/01/05/using-puppet-and-group-policy-to-manage-windows-part-2/
published: false
slug: using-puppet-and-group-policy-to-manage-windows-part-2
title: Using Puppet and Group Policy to manage Windows (part 2)
wordpress_id: 89
categories:
- Active Directory
- Puppet
---

# Introduction



I previously discussed some pros and cons of Puppet and Group Policy, most of which you could probably gather online elsewhere. But no worries, let's get to the real meat, CONFIG FILES!

As a side note, Unix file paths are not the same as Windows File paths, of course, this is pretty obvious. However, this also means the traditional \' will actually escape the single quote. Due to this, most of these config files include double quotes



## Node Manifests



For puppet we are going to use the following example for a node manifest:



<blockquote>puppet/manifests/windows-prod.pp</blockquote>




    
    node /^(addc)(\d+).domain.org$/ {
    $ip_version = 'ipv4'
    $env = 'prod'
    include role::windows_base
    include role::domain_win_promo
    }
    



In most situations, I am using numbers to represent the different Active Directory Domain Controllers. So in this example, I'll have addc1 - 10. All of the servers are designated by their intention. Previously, we used names for our servers. While fun, it was very confusing to start off at a company like this. (_side note, all my test servers are based off Nordic mythology_)

What this will do is check for any server with a name that starts with addc, has a subsequent number, and is a part of the proper domain name.



## Roles



We also need to define this role. I have a file called role.pp in the manifests folder. This file is a huge file that contains all our role definitions, it makes it easier to manage.


    
    #Windows Roles
    class role::windows_base() {
    include domain_windows
    }
    





### So what does this do?



[Puppet's definition](https://docs.puppet.com/puppet/latest/reference/lang_classes.html) on the class role states:



<blockquote>Classes are named blocks of Puppet code that are stored in modules for later use and are not applied until they are invoked by name. They can be added to a node’s catalog by either declaring them in your manifests or assigning them from an ENC. Classes generally configure large or medium-sized chunks of functionality, such as all of the packages, config files, and services needed to run an application.</blockquote>



A class states where the to reference the code. Within the brackets, there is a definition of including a module.



## Modules



A module is code that includes a bevy of information. Whether that be files, the data within those files, a template to be filled with variables. Every module has a specific layout.

I use submodules as a way to reference original module's project. This way, the original module isn't overwritten and will always be static and can be updated when more features come out for the module. I'll explain a bit more later on.


    
    git submodule add https://github.com/puppetlabs/puppetlabs-registry modules/registry
    git add .gitmodules modules/registry
    git commit -m "Adding X submodule"
    git push origin master
    git submodule update --init --recursive 
    



Depending on the amount of submodules you have, this could take a while.

So lets get to the modules:



<blockquote>puppet/modules/registry/</blockquote>



This would be the full path of the registry module and everything pertaining to it. In this example, I am only going to reference the values.pp file within this module. You can find it [here](https://raw.githubusercontent.com/puppetlabs/puppetlabs-registry/master/manifests/value.pp). However, the idea is to leave this module alone. As we are going to reference back to it by creating a custom puppet module. Lets call this "domain_registry". The intent here is to setup a base policy.

So now we have this:



<blockquote>puppet/modules/domain_registry/init.pp</blockquote>



I recently learned how to use create_resource(), which I found out is a much better at keeping the actual data within a single file. The problem is that learning create_resource can be a little confusing. I will try to do my best and explain it here.

So what does the below do?


    
    class domain_registry(
     $registrykeys = {}, 
     ) {
        create_resource(registry::value, $registrykeys)
    }
    



Here we are calling this entry "domain_registry" this is how we will reference the class in any puppet manifest.


    
    $registrykeys = {},
    



This is the name of our class. This is the identifier for the data we are going to pull from Hiera. This also indicates that the information being passed into $registrykeys is an array of data. Meaning that we are feeding multiple sets of information into Puppet. That information is stored in the Hieradata file that we will go over below.

So what does the line that starts with create_resources the create_resource is avaailable on older versions of Puppet, but, better tools are now available in Puppet such as [iteration](https://docs.puppet.com/puppet/latest/reference/lang_iteration.html).

But what is the workflow for this? As you can see we are calling registry::value. What we are doing here is telling it to take the array and have that array feed any data into the variable $registrykeys. Subsequently, use the variables within $registrykeys as data to feed into the class registry::value. So the key path would



#### Active Directory



We will also add this class so that we can setup our domain controller automatically once puppet launches automatically on the server. This should be placed in:



<blockquote>puppet/modules/domain_win_dcpromo/manifests/init.pp</blockquote>




    
    class domain_win_dcpromo(
          $dsrmencpass = {},
          $localadminpassword = {},
      ) {
        windows_ad { 'dc_promo':
          installsubfeatures        => 'true',
          restart                   => 'true',
          domainname                => 'domain.org',
          netbiosdomainname         => 'domain.org',
          dsrmpassword              => $dsrmencpass,
          localadminpassword        => $localadminpassword,
          demoteoperationmasterrole => 'false',
          uninstalldnsrole          => 'false',
        }
    }
    



The above is going to take the information we have in Hiera and set up Active Directory for us automatically.



The last module I am going to cover is the radius module. Which is important to me as the management of NPS/Radius in Windows Server is really crappy. Who in the hell thought it would be a good idea to not synchronize NPS configuration between servers. Either way, puppet can be used to do this exact function. As you can see in the hiera file below, there is an entry for npsmanagement. The only NPS Module called [puppet-npsradius](https://github.com/DevOpsFu/puppet-npsradius) by [DevOpsFu](https://github.com/DevOpsFu) I was able to find is located [here](https://github.com/DevOpsFu/puppet-npsradius). While I am glad that one already exists, I was not so glad that it came with no documentation.

The only documentation on how you will find to use this module in Hiera is how it is shown below and what we have in a our custom module folder for our personal need, you will also need to put this in the following file:



<blockquote>puppet/modules/domain_npsmanagement/manifests/init.pp</blockquote>




    
    class domain_radius (
      $radiusmanagement = {},
      $securitygroups   = {},
      $tetemplatefile   = 'domain_radius/npsradius.xml.erb',
    ) {
        class { 'npsradius':
            clients         => $radiusmanagement,
            allowedgroups   => $securitygroups,
            configtemplate  => $tetemplatefile,
      }
    }







## Heira




    
    domain_windows::windowsusernames:
        ## All users
      administratoruser:
        username: admin
        password: ENC[PKCS7,EncryptedContents]
        groups: 'Administrators'
    domain_win_dcpromo::init::dsrmencpass: ENC[PKCS7,EncryptedContents]
    domain_win_dcpromo::init::localadminpassword: ENC[PKCS7,EncryptdContents]
    domain_registry::keyentry:
        ## All NTP related settings
        ntpservers:
          key: "HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\W32Time\\Parameters"
          value: 'NtpServer'
          type: 'string'
          data: 'tick.ucla.edu'
        enablentp:
          key: "HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\W32Time\\TimeProviders"
          value: 'Enabled'
          type: 'dword'
          data: '0x00000001'
        ## Windows update
        disableautoupdates:
          key: "HKEY_LOCAL_MACHINE\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate\\AU"
          value: 'NoAutoUpdate'
          type: 'dword'
          data: '0x00000000'
        enablewindowsupdate:
          key: "HKEY_LOCAL_MACHINE\\Software\\Policies\\Microsoft\\Windows\\WindowsUpdate"
          value: 'DisableWindowsUpdateAccess'
          type: 'dword'
          data: '0x00000000'
    domain_radius::allowedgroups:
          allowedgroups:fakegrouphere
    domain_radius::radiusmanagement:
          name-of-radius-client:
            ip: 1.2.3.4
            secret: ENC[PKCS7,EncryptedContents]
    
    



From here, there are files that declare which OS the data is applied to. In this instance, I use windows.yaml in the hieradata/ folder.



This is mainly what I learned while trying to run puppet on our machines, and while I learned a lot, I still feel like I know nothing. Which I guess is the puppet way of life.

Let me know if you have any questions in the comments, and I will try to answer them as best as I can!
