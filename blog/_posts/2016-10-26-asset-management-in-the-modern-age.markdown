---
author: Andrew Doering
comments: true
date: 2016-10-26 06:00:13+00:00
layout: post
link: https://andrewdoering.org/blog/2016/10/26/asset-management-in-the-modern-age/
slug: asset-management-in-the-modern-age
title: Asset Management in the modern age
wordpress_id: 159
comments: true
tags:
- Altiris
- Asset
- Device42
- EZ Office Inventory
- JIRA
- Management
- Netbox
- Samanage
- SnipeIT
- SNOW
- Symantec
- Asset Management
---

There are a lot of tools out there that can assist with Asset Management. Dozens upon dozens, literally. I am sure that a bunch of these seem to initially been designed for multiple purposes, I personally don't believe that is a good design choice. The more solutions you add to a single utility, the more complex it gets, and the more difficult it is to manage. After starting at a recent company, I was tasked to find an asset management solution, which wasn't easy. From an IT perspective, I found exactly what I needed, but, from a financial perspective, this tool made little sense. I will be describing what I searched, the reasons for either going with or dismissing the product, and finally, what the final process was. My focus leaned towards Mac compatible software, and Windows second, as my installation base has a much broader scope with Macs.



## Asset Management Products



There are about a dozen asset management products listed that I tried. Several of which are open source, and the remainder were closed source or partially closed source. As mentioned in previous posts, I have a 0 dollar budget to usually work on. We already had a service desk ticketing system, and didn't need to migrate the entire service desk just for an asset management system. A lot of the research into different asset management solutions was done some time ago (during the beginning of 2016), so I cannot remember every exact detail of the solutions I was looking at. I inquired around in several different locations, starting with MacAdmins Slack channel, Ubuntu Forums, CDW, and of course just searching around myself. At the same time, I was also looking for a Data Center Asset/Inventory Management system, so some of my requests overlapped. Those looked at are:




    
  * [Samanage](https://www.samanage.com/)

    
  * [OCS Inventory](https://en.wikipedia.org/wiki/OCS_Inventory)

    
  * [Ralph](http://ralph.allegro.tech/)

    
  * [Snipe-IT](https://snipeitapp.com/)

    
  * [Altiris](https://www.symantec.com/products/threat-protection/endpoint-management/asset-management-suite)

    
  * [Spiceworks](https://www.spiceworks.com/free-asset-management-software/)

    
  * [EZOfficeInventory](http://www.ezofficeinventory.com/)

    
  * [JIRA](https://blogs.atlassian.com/2014/03/jira-asset-management-overview/)

    
  * [Device42](http://www.device42.com/)

    
  * [Netbox](https://github.com/digitalocean/netbox)



The above bullets link to the corresponding websites.



### EZOfficeInventory



I am starting with EZOFficeInventory as this was what we had prior to me coming on board. I honestly have to say, I hated it. As a disclaimer, our side note our account has since been suspended as we haven't paid for their product since I arrived. The interface and functionality were so restricted at our tier (if I remember right, we had the silver, but we could have had the gold tier).

[Pricing](http://www.ezofficeinventory.com/pricing) for the product seems abnormal for the device to user ratio. But the major reason is, the integrations and features they offered did not make any sense with our current in-place systems. Our primary SAML system is Okta, and while they do integrate with that, they only integrate at the Platinum level or higher. They integrate with Zendesk, Dropbox, and a few other services. Along with that, their interface seems second rate.



### SpiceWorks



After looking at SpiceWorks, I was told it was not an option at all, mainly due to privacy concerns on their side. So out goes that option. Plus while I was looking, I noticed that they put advertisements in a product that they are wanting to "sell", which seems ridiculous. I am fully aware that they offer their services for free, but that is part of the reason why they were not an option. I don't want any of my companies computers having aggregated data collected on and being sent to them. Plenty of reviews exist on the internet, [here](https://www.trustradius.com/reviews/spiceworks-2014-06-20-09-29-45) [are](http://www.itsmdaily.com/spiceworks-review/) [a](http://www.pcmag.com/article2/0,2817,2345210,00.asp) [few](http://www.techrepublic.com/blog/product-spotlight/review-spiceworks-network-management/).



### Altiris



Lets be honest, I want to stay the hell away from Symantec products. Especially Altiris. I used this product at a previous workplace, and it was just terrible. The agent would have problems all the time. Causing an inordinate amount of pain when attempting to uninstall to repair the problem. Plus, you have to buy add-on items to use the product to it's fullest potential, and Symantec buries it's roots deep into the OS. It also means that we will be constantly bothered to interact or expand on other services from their sales people, and as of now, I have too many other things to worry about then a Symantec sales person. Support is also a night mare with Altiris.



### Snipe-IT



Snipe-IT is open source software, and has a pretty active dev team. Their [Github page](https://github.com/snipe/snipe-it) is kept up to date, and releases are at least twice a month (beta and/or stable). They also have a [demo available](https://snipeitapp.com/demo) with close to full administrator capability. Initially, I poked around the demo, and was off-put by the user interface. During version 2.X, the user interface wasn't awesome. It also seemed convoluted at the time, however, that is most likely because of the amount of fake data and oddness in the demo environment.




    
  * A way to initiate users into the system via LDAP for easy checkout

    
  * A way to export reports from the system for auditing purpose

    
  * Import any data via CSV file

    
  * Has internal alerting for low inventory

    
  * Has a way for user self-service (if an employee wanted to check out an asset)

    
  * Informs of IT via Slack or other chat system via a webhook for events



All of this sounded great. After a month, I checked back up on Snipe-IT and noticed that they had released a new version of the software (version 3.X). The new version of the software, appeared to be much better and allowed for greater control of assets and better customization. It also helps that the developers are quite attentive and the software is easy to manipulate and tweak to what we would like to use.



### Ralph



This was intended to be for the data center. Ralph, like Snipe-IT is also open source software, with an active development team as well. Ralph also features a [demo](http://ralph.allegro.tech/) of their software. Again, I initially looked around the demo before testing out the installer, it led me to originally believe it would work much better for a data center inventory tool, over the financial asset management side. It had some really nice things about it, namely the very extensive [Data Center visualization](https://ralph-ng.readthedocs.io/en/latest/user/quickstart/#data-center-visualization). A video of which can be seen below from one of the devs.



This was pretty cool, that free software was starting to offer this functionality, as it would have made the data center team incredibly grateful. It also seemed to provide similar concepts that Snipe-IT did in terms of the bullets above. Unfortunately, Ralph does not provide certain aspects of IP Management that we needed to continue with this product. A lot of functionality that was in Ralph is exactly what we were looking for. It also, did not have a reliable agent for devices yet and was currently in the works. It was great but not what we needed at the moment.



### Device42



This tool also showed promise, it did most everything we needed in a datacenter. They even provide a [discovery script](https://github.com/device42/nix_bsd_mac_inventory) to get started in terms of their API. There are problems with this script though, namely that if you follow proper system security, each machine should have it's own password. This provided script relies on using a single machine to SSH to to pull data from other systems about what you want entered automatically into their service. I had some problems convincing the team that they might need to create their own python script to efficiently run, mainly because our team is already strained on resources, we expect something like this to work out of the box for the most part. If you are providing a service like this, the script should work smoothly and without having to deal with a bunch of stuff to get it working in a proper way. It's also not explicitly said, but, there wasn't any information provided that these passwords could be read in anything other than plain text in their provided form. This can be seen [here](https://github.com/device42/nix_bsd_mac_inventory/blob/master/inventory.cfg.sample) and also attached here if this is updated in the future:

Commit 4da2edd on Aug 19


    
    [settings]
    # base_url points to your Device42 server
    # For example: base_url = https://192.168.1.50
    base_url   =  https://
    # Device42 username and password
    username   =  admin
    secret     =  adm!nd42
    




    
    [credentials]
    # If using key file instead of username and password, enter 'True' for use_key_file setting and
    # path to the private key file. For example:
    # use_key_file = True
    # key_file = /home/user/.ssh/id_rsa
    use_key_file = False
    key_file     =
    # You can use as many USER:PWD combinations as you like separated by commas.
    # TIP: put more frequently used combinations at the beginning of line
    # For example: credentials = user:P@ssw0rd,root:P@ssw0rd
    # If using key file, instead of user:password combination use just username
    # For example: credentials = root
    credentials   =
    



Device42 also has a similar problem in terms of them being unable to provide the necessary information that we need for IP Management in terms of specific protocols that we would like to keep track of. Again a requirement. So this option was also thrown out the window.



### Jira



After looking at what seems like the only option in JIRA, it became immediately clear that this would be an incredibly manual process. There didn't seem to be a very efficient workflow for this to work for us. My company already sometimes pushes our Jira cloud instance to it's max. Since the main purpose of JIRA is not an asset management system, we also have to take this into consideration. There are better tools out there anyways for asset management, so we decided to take a pass on JIRA.



### OCS Inventory



OCS Inventory has a [demo](http://demo.ocsinventory-ng.org/index.php) just like most others, which is great. The interface looks manageable, it is somewhat streamlined and looks "up-to-date". It provides IP and SNMP scanning for inventory which was a nice addition. However, I don't like relying on IP and SNMP scans, I find them not always reliable in the information that is provided. In addition devices move around all the time, relying on IP just doesn't work for scans. While OCS Inventory supports Mac OS X, there are no OS X systems in their demo and the BSD systems that are there aren't displaying information that we would like to see. I also didn't really get good vibes from the software. I know it is in use in dozens on dozens of deployments around the world, but, when I looked at it, it felt like it was "stuck".



### Samanage



Samanage uses OCS Inventory for it's Mac and Linux discovery tool. I am not sure of their Windows tool platform, but it didn't appear to be based on OCS Inventory. While their interface was nice, it also had issues properly collecting information by default. For instance, it couldn't properly distinguish between the hostname of the Mac and bound Active Directory name reported by dsconfigad. Which will cause identification issues when trying to locate the computer within our AD system. Most of this tool already did something that we were looking for in terms of a service desk, and a full inventory system (we are using Munki for Macs, and still looking at options for Windows). Again a lot of repeat functionality that isn't needed. It didn't seem like it would reliably work for us for the data center or for the endpoint client.



# The Golden Child(ren)





## Snipe-IT



In the end, I selected Snipe-IT for a user-hardware/financal asset management system for a few reasons, which are mostly described above, but will summarize again.




    
  * Open Source Software

    
  * Easily customizable in terms of the database and additional web components

    
  * Can integrate with [Munki](https://github.com/munki/munki) and [MunkiReport-PHP](https://munkireport.github.io/munkireport-php/) after some tweaking on both sides

    
  * Can easily accommodate CDW Purchase Reports

    
  * On the Finance side, it can keep track of all asset information (order number, price of item, purchase information, etc)

    
  * It can assign licenses per asset/person.

    
  * Allows for group-specific settings that allow certain team members (Finance, HR, InfoSec, and of course IT) to pull reports for specific reasons (auditing, terminated employee hardware, etc).

    
  * Didn't have additional bells and whistles we did not need.





### Hosting



Snipe-IT allows for two hosting methods, self-hosted solution or cloud-host solution. Either solution was ideal for us, as hosting it ourselves would allow for the seamless LDAP connection, and hosting it outside would provide for ease of management. In the end, we decided to host it on-prem, as the need for a cloud hosted version wasn't needed and we only have one main office currently and the security risk on the application itself was minimal enough.



### Installation



Installation was incredibly easy. It also helps that the [documentation](https://snipe-it.readme.io/docs/getting-started) is pretty in-depth and explains mostly anything you need to do. There is also a shell script to install all the necessary components. I would recommend setting everything by following the instructions in their documentation. It literally as simple as possible. My only recommendation is to use nginx instead of Apache (which is the default).



#### snipeit.sh method



This will work as of Ubuntu 16.04.1




    
  1. Lets start off at your home directory, and create and change into the directory specifically for SnipeIT.

    
    cd ~ && mkdir snipeit && cd snipeit




    
  2. Download the [install.sh](https://github.com/snipe/snipe-it/blob/master/snipeit.sh)

    
    wget https://raw.githubusercontent.com/snipe/snipe-it/master/snipeit.sh




    
  3. Make the file executable

    
    chmod 744 -v snipeit.sh




    
  4. Run snipeit.sh as sudo

    
    sudo ./snipeit.sh




    
  5. You will be prompted for several questions here which you will want to answer with any future changes in mind. For example if you want to manually setup a password or have SnipeIT generate a password for you, setting the FQDN of the server, etc.

    
  6. You are finished with the snipeit.sh method of install and are no on to post installation stuff!





### Post-Installation



So far I have found a few things that are issues with Snipe-IT:




    
  * The License Key field in the Database is capped at 252 characters. [Github feature request](https://github.com/snipe/snipe-it/issues/2535) already submitted and closed. It is also possible to manually edit the database yourself using the following command once you have logged into your mysql database:




    
    use snipeit;
    ALTER TABLE licenses MODIFY serial VARCHAR(2048);



While this is no longer needed with the previously mentioned feature request, for those that have not upgraded the command is still necessary. This will help resolve longer license keys like 3T MongoChef. If you ever need to reset back to the default column_type, the default varchar length is 255. So just replace 2048 with 255.




    
  * Switch from Apache2 to nginx for a bit better response times on the site when load is applied. There is a helpful information [here](https://snipe-it.readme.io/docs/linuxosx#using-nginx-and-php-fpm). However, with Ubuntu 16.04.1 and any other PHP7.0 package, you will need to tweak a few (or one) things.






    
    server {
     listen 80;
     server_name domain.org;
    
    return 301 https://$server_name$request_uri;
    }
    
    server {
     listen 443 ssl;
     server_name domain.org;
    
    ssl_certificate /path/to/your.crt;
    ssl_certificate_key /path/to/your.key;
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-RC4-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:RC4-SHA:!aNULL:!eNULL:!EXPORT:!DES:!3DES:!MD5:!DSS:!PKS;
    ssl_session_timeout 5m;
    ssl_session_cache builtin:1000 shared:SSL:10m;
    
    root /Users/youruser/Sites/snipe-it/public/;
     index index.php index.html index.htm;
    
    location / {
     try_files $uri $uri/ /index.php$is_args$args;
     }
    
    location ~ \.php$ {
     try_files $uri $uri/ =404;
     fastcgi_pass unix:/var/run/php7-fpm-www.sock;
     fastcgi_index index.php;
     fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
     include fastcgi_params;
     }
    }
    
    



Notice that the fastcgi_pass line has changed from the original in the readme.




    
  * Setup and force SSL. There is ABSOLUTELY NO REASON to not use SSL at this point in time. If you can't generate a trusted certificate, create a self-signed certificate and update the ssl_certificate and ssl_certificate_key paths in the above config to the appropriate place to where your SSL Certificate resides.

    
  * Create Locations in SnipeIT. This is not a pre-populated field, and if you have more than one office location or employees in the field, it is super beneficial for you to create the proper location fields you need before you dive into anything else.

    
  * Create a snipeit service account for binding to LDAP if required.

    
  * Use LDAP to import your current employees, rather than doing them by hand. Snipe-IT has nifty LDAP feature. It also integrates into the Location feature above, and is essential to properly importing users from LDAP.

    
  * Modify the Cookie settings in the .env file. The way that the default cookie settings are specified in the .env file are not necessarily the most secure. My recommendations would be to set it up in a way similar to this:

    
    <span class="cm-comment">#--------------------------------------------</span>
    <span class="cm-comment"># OPTIONAL: SESSION SETTINGS</span>
    <span class="cm-comment"># --------------------------------------------</span>
    <span class="cm-def">SESSION_LIFETIME</span><span class="cm-operator">=</span><span class="cm-number">12000</span>
    <span class="cm-def">EXPIRE_ON_CLOSE</span><span class="cm-operator">=</span><span class="cm-atom">true</span>
    <span class="cm-def">ENCRYPT</span><span class="cm-operator">=true</span>
    <span class="cm-def">COOKIE_NAME</span><span class="cm-operator">=</span>snipeit_session
    <span class="cm-def">COOKIE_DOMAIN</span><span class="cm-operator">=yourdomain.org</span>
    <span class="cm-def">SECURE_COOKIES</span><span class="cm-operator">=true</span>


These differ from the default settings which have the cookie never expire, the cookie contents are not encrypted, and secure cookies is set to false.

    
  * When you are absolutely ready, and not before, enable email alerts. There is no reason to alert end-users or your IT staff when you are trying to either test this out or put it into production. So when you are ready to have all employees use this to request assets, enable email alerts at that time. When you are ready for this change or add to your .env file:

    
    <span class="cm-comment"># --------------------------------------------</span>
    <span class="cm-comment"># REQUIRED: OUTGOING MAIL SERVER SETTINGS</span>
    <span class="cm-comment"># --------------------------------------------</span>
    <span class="cm-def">MAIL_DRIVER</span><span class="cm-operator">=</span>smtp
    <span class="cm-def">MAIL_HOST</span><span class="cm-operator">=smtp.server.com</span>
    <span class="cm-def">MAIL_PORT</span><span class="cm-operator">=</span><span class="cm-number">587</span>
    <span class="cm-def">MAIL_USERNAME</span><span class="cm-operator">=</span>YOURUSERNAME
    <span class="cm-def">MAIL_PASSWORD</span><span class="cm-operator">=</span>YOURPASSWORD
    <span class="cm-def">MAIL_ENCRYPTION</span><span class="cm-operator">=tls</span>
    <span class="cm-def">MAIL_FROM_ADDR</span><span class="cm-operator">=YOURUSERNAME@domain.org</span>
    <span class="cm-def">MAIL_FROM_NAME</span><span class="cm-operator">=</span>Snipe-IT




    
  * I personally setup Fail2Ban on all my linux servers with very strict settings for banning. This may not be appropriate in your environment. I like to try and keep things as secure as I possibly can. So this is part of that.





## Netbox



After some input from the Networking team, we decided to go with Netbox for our data center. We have not yet set up or implemented it however, so I can't really go over our setup or anything similar to it.



## Wrap Up



I hope you really enjoyed this! I know there is some weird formatting in here, wordpress was having some awful issues adhering to their own standard in terms of how to deal with markdown.
