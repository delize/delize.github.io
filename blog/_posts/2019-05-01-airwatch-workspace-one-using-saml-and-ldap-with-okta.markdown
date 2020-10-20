---
author: Andrew Doering
comments: true
date: 2019-05-01 21:15:00 PT
excerpt: This is a good first read if you want to delve into SAML and WS1 configurations. Please see the updated post for a more up to date and more functional guide. 
layout: post
link: https://andrewdoering.org/blog/2019/05/01/airwatch-workspace-one-using-saml-and-ldap-with-okta/
slug: airwatch-workspace-one-using-saml-and-ldap-with-okta
title: Airwatch/Workspace ONE - Using SAML and LDAP with Okta
wordpress_id: 514
comments: true
tags:
- Okta
- Workspace One
- Airwatch
- LDAP
- LDAP Interface
categories:
- Identity Access Management
- Mobile Device Management
  
permalink: /blog/:year/:month/:day/:title/
robots: all, noarchive
---












**UPDATE 12/22/2019** - Please see[ this post](https://andrewdoering.org/blog/2019/12/22/workspace-one-okta-users-and-groups-working-with-dep-enrollment/) for a more up to date and more functionaly guide. It is good to read this first, before doing the other one however to get a better understanding of our initial decisions.



- [Precursor](#precursor)
  - [Software Requirements](#software-requirements)
  - [Initial Expectations](#initial-expectations)
- [Workspace One - Directory Services Integration](#workspace-one---directory-services-integration)
- [Okta Profile Mapping](#okta-profile-mapping)
- [Airwatch - User and Groups Directory Integration](#airwatch---user-and-groups-directory-integration)
  - [Users](#users)
  - [Groups](#groups)
- [OKTA - SAML App for Airwatch](#okta---saml-app-for-airwatch)
  - [SAML Integration](#saml-integration)
  - [sDP Bookmark Application](#sdp-bookmark-application)
- [Airwatch - SAML Settings](#airwatch---saml-settings)
- [Room for Improvements](#room-for-improvements)



In this writeup, I will go over the configuration settings that I used, and worked with Airwatch/Workspace ONE Professional Services and Support  to get Airwatch up and working within ThousandEyes. Initially, before writing this guide (which I have admittedly been putting off for a while - since May when we did our deployment of the service), the documents that were available on VMware's website, were mostly incorrect. This might have been fixed, however I have not gone back to validate any of their documentation. Once we spend the time to do this, I will update either this page, or create a new posting.







## Precursor







### Software Requirements







  * Airwatch Cloud Connector (optional)
  * Okta Ldap Interface
  * Okta (duh)
  * Airwatch / Workspace One (duh)






### Initial Expectations







ThousandEyes has not yet made the transition to a full cloud based Directory System, we are still using Active Directory On-Premise. However, all systems that have been deployed going from 2019 - forward, need to or should be setup in a compatible way so that when we make the full cloud transition, it is seamless. As such, we assume here that you have or are using:







  * Active Directory On Premise (as Profile Master in Okta)
  * Okta (as secondary)
  * Intent to put BambooHR as Profile Master later on
  * Intent to move Active Directory On Premise to Azure AD






We actually tried using VMware's Identity Manager to perform SAML, but at the time, the directions did not allow configuration after following the guides provided by Okta or VMware. Because they said this was the easier way, and unfortunately when asking them - they wouldn't bother helping us with the correct way. In addition, originally their documentation that was listed here, was incredibly wrong. It seems like in the time that I have taken to put off writing this guide, they have updated their documentation to show the correct way to connect Okta as an LDAP service. 







Documentation [listed here](https://kb.vmware.com/s/article/2961230?lang=en_US), gives you a proper configuration guide. However, at the time of originally writing this guide, the documetation was incorrect and did not work.   
  
With that said, I would also highly recommmend to ignore [this guide](https://www.okta.com/sites/default/files/Okta-3rd-Party-UEM-Interop_Workspace-ONE.pdf), it is outdated for newer versions of Workspace One.







So with this in mind, let me explain how ThousandEyes did this:







## Workspace One - Directory Services Integration







Setting this portion up is quite easy with the way that the support team and consultant recommended. Unfortunately, the easy way is not the best way, and you should be able to get assistance from support easily on how to do this. However, this is how ThousandEyes has performed this:





![LDAP Connection Settings](/assets/blog/2019/11/image.png)





To edit any of the settings for a currently working directory, just replace the strings that are in capital letters below with yours.






    
    <code>Directory Type: Other LDAP
    Server: YOURDOMAINHERE.ldap.okta.com
    Encryption Type: SSL
    Verify SSL Certificate: Check the box
    Bind Authentication Type: Basic
    Bind User Name: uid=YOURSUPERUSERHERE,dc=YOURDOMAINHERE,dc=okta,dc=com
    Bind Password: YOURSECRETHERE
    Domain: YOURDOMAINHERE.okta.com</code>







This should allow you to verify or test the connection below. Once the LDAP Settings are working, move on to the User tab.







## Okta Profile Mapping







Because we want to integrate this with Azure later on, and Active Directory, we need to map some of our existing attributes from Active Directory and push them into Okta.







First, go here:  [https://YOURDOMAIN-admin.okta.com/admin/universaldirectory](https://YOURDOMAIN-admin.okta.com/admin/universaldirectory) and select the Profile button located below





![Profile Editor](/assets/blog/2019/11/image-4.png)





Click the add attribute button





![Add Attribute](/assets/blog/2019/11/image-5.png)





Use the the following settings for the Attributes





![](/assets/blog/2019/11/image-2.png)



![](/assets/blog/2019/11/image-3.png)





Now, lets go to Active Directory's mapping, select your domain to setup the mapping push. You want to push from the Active Directory Domain to Okta.





![](/assets/blog/2019/11/image-6.png)





## Airwatch - User and Groups Directory Integration







### Users







Setting up the users panel





![](/assets/blog/2019/11/image-1.png)





While VMware's documentation states to use mail=, I found that for the ease of the employee, using their shorthand account name is far easier for the experience - specifically for macOS. With Windows, this change may cause problems.







Drop the advanced menu down and select the following features





![](/assets/blog/2019/11/image-7.png)





We came to the conclusion of using the following settings for the advanced section of the user system





![](/assets/blog/2019/11/image-8.png)





The reasons for these attributes being used are based directly on the attributes that are listed in Profile attributes in Okta, as well, as the ldapsearch tool based on the information below:






    
          ldapsearch -H ldaps://thousandeyes.ldap.okta.com -b "dc=thousandeyes,dc=okta,dc=com" -D "uid=adoering@thousandeyes.com,dc=thousandeyes,dc=okta,dc=com" -W -x "(sn=Doer)"
          # extended LDIF
          #
          # LDAPv3
          # base <dc=thousandeyes,dc=okta,dc=com> with scope subtree
          # filter: (sn=Doer)
          # requesting: ALL
          #
          
          # atesting@thousandeyes.com, users, thousandeyes.okta.com
          dn: uid=atesting@thousandeyes.com,ou=users,dc=thousandeyes,dc=okta,dc=com
          objectClass: top
          objectClass: person
          objectClass: organizationalPerson
          objectClass: inetOrgPerson
          uid: atesting@thousandeyes.com
          uniqueIdentifier: REMOVINGFROMHERE
          organizationalStatus: ACTIVE
          givenName: Andrew
          sn: Doer
          cn: Andrew Doer
          mail: atesting@thousandeyes.com
          mail: atesting+testing@thousandeyes.com
          AD_ObjectGUID: REMOVINGFROMHERE
          sAMAccountName: atesting
          title: IT Tester
          department: Engineering
          
          # search result
          search: 2
          result: 0 Success
          
          # numResponses: 2
          # numEntries: 1







When we look  at this, we see there are a few differences in Okta's LDAP directories compared to Active Directory. So those adjustments are made into the above attributes. A few of these attribute changes are not mentioned in VMware's documentations. 







### Groups







With groups, we came to the conclusion that the following attributes were required for use within Airwatch:





![](/assets/blog/2019/11/image-10.png)



![](/assets/blog/2019/11/image-9.png)





Unfortunately, even with this setup - Airwatch does not support importing users from the groups itself. While this might be an issue only we have, I have not found a way around getting the users imported - even with the proper settings. While we have opened up a support ticket for this (and even had confirmation in the past that this was a bug), 6 months later it is still not fixed. See the Room for Improvements section for a feature request using VMware's new feature request option.







## OKTA - SAML App for Airwatch







### SAML Integration







Create a custom SAML Application in Okta. 





![](/assets/blog/2019/11/image-11.png)





Attach a photo if you would like, however, this application will mainly be hidden in the application dashboard. Go ahead and click next.







Below you will see a heavily redacted SAML setup. Sorry, but as these are all private links, I can't post them. However, I have





![](/assets/blog/2019/11/image-12.png)





However, you will see a modified version of the links below where you can change the wording that is in all capital letters.






    
          Single Sign On URL: https://YOURDSSERVERHERE.awmdm.com/IdentityService/SAML/AssertionService.ashx?binding=HttpPost
          Requestable SSO URLs, Index
          https://YOURDSSERVERHERE.awmdm.com/IdentityService/SAML/AssertionService.ashx, 2
          https://YOURDSSERVERHERE.awmdm.com/MyDevice/SAML/AssertionService.ashx, 5
          https://YOURDSSERVERHERE.awmdm.com/DeviceManagement/SAML/AssertionService.ashx, 8
          https://YOURCNSERVERHERE.awmdm.com/AirWatch/SAML/AssertionService.ashx, 11
          https://YOURDSSERVERHERE.awmdm.com/Catalog/SAML/AssertionService.ashx, 14
          Audience URI: SOMETHINGHERE
          
          
          Attribute Statements
          Name, NameFormat, Value
          uid, Unspecified, user.login
          objectGUID, Unspecified, user.AD_ObjectGUID
          uniqueIdentifier, Unspecified, user.getInternalProperty("id")







You might be curious about the last piece here. This is a pretty way to use the Okta Expression Language to get the uniqueidentifier property that we were using previously as a way to send the string without it being an visible attribute to the user. 







Download the metadata file that is generated.







### sDP Bookmark Application







Becuase Airwatch in this way only provides SAML when initiated from the software layer. Because of this, we will be creating a bookmark application to directly go to the URL that allows for SAML login provided by the Group ID within the instance.






    
        https://YOURCNSERVERHERE.awmdm.com/AirWatch/Login?GID=YOURGROUPIDHERE







From there, simply add a photo, my recommendation would be to use this photo:  
![](https://images.app.goo.gl/FHoBaE4A912jCRmg9)







## Airwatch - SAML Settings







Go to Enterprise Integrations > Directory Services > SAML. Take the Metadata file generated and upload it to the Import Identity Provider Settings area. 





![](/assets/blog/2019/12/image-1-1024x841.png)





The fields should populate like so. 







From here, you should be ready to go with logging into the Bookmark application you made. 







## Room for Improvements







There are various problems with this integration, and even when setting up with proserv, this did not come out the way I would have liked. 







  * Airwatch & Azure integration does not work/is not configured.  
Unfortunately, proserv was not super helpful with this in prepping for Azure integration for the future.  The only requirement that was checked on was for 
  * Our deployment was guided mainly towards macOS  
This causes a disparity between what we need and what we want. We requested that we setup both macOS and Windows configurations here. 
  * Airwatch's ProServ consultants can vary wildly in knowledge regarding the application.  
This was the case with ours, we switched consultants three times. We required the use of multiple people to get our Okta setup working, this was multiple weeks.
  * Groups are not usable in this integration  
Even with the required setup, using Airwatch's documentation, group membership listing does not work, groups do import but no user listings will show. There is a [Feature Request](https://wsone-uem.ideas.aha.io/ideas/UEMCP-I-291), as well as support case open for this.
  * After writing this, I looked up Airwatch's documentation - it is vastly improved. It looks like the documentation can finally be used as a mechanism to implement features and integrations.


