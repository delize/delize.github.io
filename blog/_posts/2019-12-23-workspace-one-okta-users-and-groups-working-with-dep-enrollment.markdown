---
author: Andrew Doering
comments: true
date: 2019-12-23 07:03:05 PT
excerpt: This guide covers how to setup member and group importing, the correct way, in Airwatch/Workspace One using Okta as an LDAP backend.
layout: post
link: https://andrewdoering.org/blog/2019/12/22/workspace-one-okta-users-and-groups-working-with-dep-enrollment/
permalink: /blog/:year/:month/:day/:title/
slug: workspace-one-okta-users-and-groups-working-with-dep-enrollment
title: Workspace One & Okta - User(s) and Group(s) working with DEP Enrollment
comments: true
tags:
- airwatch
- DEP
- groups
- macOS
- okta
- one
- users
- vmware
- workspace
- workspace one

blogsingle-backgroundimage: https://images.wallpapersden.com/image/download/huawei-4k-stock-abstract_66336_1920x1080.jpg
img_bg: https
img_bg_alt: Placeholder Alt text
img_blog: /assets/img/2019-12-22-post/okta_ws1_interop.jpg
img_alt: Image about Okta and Workspace1 Interoperability. Intro image.
robots: all, noarchive
---


- [Introduction](#introduction)
  - [How this document is structured](#how-this-document-is-structured)
- [Why is Okta LDAP and WS One UEM Intergation important?](#why-is-okta-ldap-and-ws-one-uem-intergation-important)
- [Software Requirements](#software-requirements)
- [Okta - Adding User Attributes](#okta---adding-user-attributes)
- [Workspace One - Directory Services > Server Configuration](#workspace-one---directory-services--server-configuration)
- [Workspace One - Directory Services > User Mapping](#workspace-one---directory-services--user-mapping)
- [Workspace One - Directory Services > Group Mappings](#workspace-one---directory-services--group-mappings)
- [Okta - Limitations to be aware of](#okta---limitations-to-be-aware-of)
- [Workspace One - Test your setup](#workspace-one---test-your-setup)
- [Workspace One - Add Group](#workspace-one---add-group)
- [Workspace One - DEP Profile](#workspace-one---dep-profile)
- [Okta - SAML Application](#okta---saml-application)
- [Our room for improvement](#our-room-for-improvement)
- [Comments, Questions?](#comments-questions)



## Introduction







If you have not yet read [the previous writeup about Airwatch and SAML,](https://andrewdoering.org/blog/2019/05/01/airwatch-workspace-one-using-saml-and-ldap-with-okta/), I would highly recommend reading this before finishing the setup, this page will pick up on where we left off on the Room for Improvements section. Namely, getting user imports, and Okta groups to have successful lookups as well as member imports.  
  
I also have to apologize, as this blog post was written on plane wifi, and as we all know... plane wifi is awful. I coincidentally, had to VNC to a differenct machine to even get into my blog, as the connection would time out.  So I had to write most of this from memory and without photos as a step by step guide on how to do this configuration.







**Note:** that everything listed here is not for the Workspace One Identity Manager, but solely for LDAP Settings







**Note 2:** As of writing this, this has only been tested in a Feature Staging environment, as I did not want to make a change over the weekend, nor did I want to make such a drastic change into our directory services over the holiday break without the time to properly test. So this has been tested in our feature stage but not in production. However within the feature stage, I was able to DEP enroll devices. Automatically add the users into groups, and import group objects and user objects without issue, and allow SAML.







**Note 3:** “After writing this, I looked up Airwatch’s documentation – it is vastly improved. It looks like the documentation can finally be used as a mechanism to implement features and integrations.” This quote is taken from my first blog post. Airwatch’s documentation still needs improvement. Their directions for this were completely wrong, and I spent over a month (or what seemed like it) going back and forth with support. In the end, it was more of dumb luck getting this to work, as I eventually got on a call with Okta & Airwatch Support, and neither of them unfortunately knew what specific attributes needed to be used either. 







**Note 4**: This is working as of January 1st 2019, I can’t guarantee that it will continue to work after this date. (Though I don’t see why it wouldn’t).







### How this document is structured







I will go over the prerequisites first, each title should will be either the task being performed, or in Airwatch’s case, the settings page which is being viewed (and also the task being performed). The document will black out specific or private information to the company that I am not allowed to share - if you noticed from the original blog post, not a lot was edited out for you. 







I use Airwatch and Workspace One synonymously. Sorry about that. 







As Monty Python skits would say “Get on with it”… let us do so.







## Why is Okta LDAP and WS One UEM Intergation important?







Having a lack of groups causes us some issues when assigning smart groups, applications, policies, etc. Because of this, we were pretty much limited to an all or nothing approach or specific lookups when trying to determine things like all devices over macOS 10.15, or a specific subset of macOS devices.







I also know that several people have been struggling to get this working properly, and Airwatch/VMware has been less than helpful for the most part to those users as their tickets are still open. The same exists on the Okta side, as it was discussed there during our Okta meetings with them.







Slow but sturdy steps in our race to achieve greatness with Workspace One!







## Software Requirements







  * Airwatch Cloud Connector (optional) 
  * Okta Ldap Interface
  * Okta
  * Airwatch / Workspace One






##  Okta - Adding User Attributes 







Please note, that I copied this verbatim from the previous blog post. 







Because we want to integrate this with Azure later on, and Active Directory, we need to map some of our existing attributes from Active Directory and push them into Okta.







First, go here:  [https://YOURDOMAIN-admin.okta.com/admin/universaldirectory](https://YOURDOMAIN-admin.okta.com/admin/universaldirectory) and select the Profile button located below





![](/assets/blog/2019/11/image-4.png)





Click the add attribute button





![](/assets/blog/2019/11/image-5.png)





Use the the following settings for the Attributes





![](/assets/blog/2019/11/image-2.png)



![](/assets/blog/2019/11/image-3.png)





Now, lets go to Active Directory's mapping, select your domain to setup the mapping push. You want to push from the Active Directory Domain to Okta.





![](/assets/blog/2019/11/image-6.png)





## Workspace One - Directory Services > Server Configuration







Please refer to [this link](https://andrewdoering.org/blog/2019/05/01/airwatch-workspace-one-using-saml-and-ldap-with-okta/#3-workspace-one---directory-services-integration) if you would like to go into more depth about how to setup this section.







Our Directory Services type setup is the same as last time, I will post it again here for your convince.  
To reiterate, you need to have a user that is either ReadOnly or SuperAdmin. The preferred method would be ReadOnly, as if your account credentials are scraped somehow from Airwatch, there will be minimal access potentially given up. Sure they will have read only access to everything but they won’t be able to do anything except read the config (still bad, but less of a risk). While you can’t control the risk of your password being dumped from Airwatch (that risk is on Airwatch) you can definitely device the risk of control given out (your access level of your BIND User). I will not go 





![](/assets/blog/2019/11/2019-11-28_12-39-35.jpg)




## Workspace One - Directory Services > User Mapping







Now here is where we get to the fun part. This is where both Okta’s and VMware’s documentation are incredibly wrong. 







For the User Mapping you will want to fill out the following attributes, with their corresponding mappings, setting everything else to the Airwatch Default.





![](/assets/blog/2019/12/image-3.png)




    
    Object Identifer: uniqueIdentifier
    User Name: uid
    Member Of: memberOf #leave default
    Full Name: cn
    Display Name: cn
    First Name: givenName #leave default
    Middle Name: middleName #leave default
    Last Name: sn #leave default
    Email Address: uid
    Email Username: sAMAccountName
    Mobile Phone: mobile #leave default
    Phone Number: telephoneNumber #leave default
    Distinguished Name: dn
    User Principal Name: uid
    Status: UserAccountControl #leave default
    Lockeout Time: lockoutTime #leave default
    Object Category: objectCategory #leave default
    Last Modified: whenChanged #leave default
    Binding Attribute: (blank) #leave default, or remove mapping
    Employee ID: employeeID
    Cost Center: (blank) #leave default, or remove mapping
    Manager Distinguished Name: managerName #leave default
    







The `#` and to the right are listed as comments, the mappping should be the value immediately after the `:`. This should get you a working configuration, assuming you followed the Okta Mapping from the previous blog post, otherwise replace `sAMAccountName` with some other attribute from Okta that you have created. I will get into why we are using this specific attribute later on.







Now, save the following settings and then scroll up to the top of the page. Now the important piece here is that `User Search Filter` will change automatically to something that we don’t want. It is a little counterintuitive but, when it does this automatic change, it also removes the ability for your group lookup to work properly. I found that deleting the first entry of this and setting it to `($(uid={EnrollmentUser}))` works much better than what is stated here. Your page should end up looking like this:





![](/assets/blog/2019/12/image-6.png)





You will need to save the mapping again, and then switch to the groups tab.







## Workspace One - Directory Services > Group Mappings







I was so very close in the original blog post for our group mappings, I only messed up one attribute value. Please see the correct settings below:





![](/assets/blog/2019/12/image-7.png)




    
    ```
    ObjectIdenfiter: uniqueIdentifier
    Name: cn
    Member: uniqueMember
    Common Name: cn #leave default
    Member Of: memberOf #leave default
    Group Object Class: objectClass
    Organizational Unit: ou #leave default
    Organizational Unit Object Class: objectClass
    ```







You may need to set the value of `Maximum Allowable Changes` to something higher than 10, maybe something like 100 or 500. Depends on some of the initial imports you will be doing.  Save the group settings, you should be done with this page.







## Okta - Limitations to be aware of







Again, as we are using the LDAP Interface, this interface will ONLY respond with Okta mastered groups. Active Directory Groups that have been pushed to Okta will NOT respond to the LDAP interface lookup. Unfortunately, I don’t believe Okta has any plans on changing this in the future.







## Workspace One - Test your setup







Once you have saved the entire configuration (both members and groups) click the test connection. You should be prompted with Workspace One’s new LDAP/Directory Services testing tool. I wish they had this when we originally set stuff up, as it makes troubleshooting the mappings a lot easier.







If you test a member, you should see something similar to the following:





![](/assets/blog/2019/12/image-8.png)





If you test a group, you should see something similar to the following:





![](/assets/blog/2019/12/groups.png)





From here, if everything looks similar to the photo above, you should be good to go.







## Workspace One - Add Group







Again, remembering that the limitation here is that Okta can only deliver groups created within Okta and not from external sources, create a group (or use an existing group) with a select few test users you want use. There is  also a limitation that if the user does not exist, they will not be created upon the sync of the group.







I will create a group in Okta called `iam_airwatch_test_group`





![](/assets/blog/2019/12/image-10.png)





So we see that there are two users within the group here. If I then switch back to airwatch and search for the group we get the following:





![](/assets/blog/2019/12/image-11.png)




So from here, lets say I add the group, my atesting account already exists in the user directory, so we have a single user in the group as of now, and then I go add our second testing user to Airwatch's directory service, the user is automatically added into the group and the group syncs up and reports two users in Airwatch as group members:





![](/assets/blog/2019/12/image-12-1024x68.png)





There you have it, group membership working properly!







## Workspace One - DEP Profile 







Now, I personally don’t want our user’s usernames being user@thousandeyes.com, and I noticed that Workspace One is quite buggy with version 1910 and the DEP Enrollment profile for macOS 10.14+ devices (specifically, the autofill settings when not using custom enrollment). If you make any changes to the above settings, like we just did, I have HAD to change the settings on this page as a result for them to take effect on all new DEP Devices.  For the most part, all I would do is just toggle the settings back and forth for the Autofill options, or your hidden administrator for them to repopulate, however if that does not work, I will show you how we have it configured.







How we have this configured at ThousandEyes: 






![](/assets/blog/2019/12/image-9.png)





So because we want the user’s account name to be the email prefix, we are relying on the `sAMAccountName` that we are passing from Okta to Workspace One's `{EmailUserName}` attribute. You could also pass this as a custom attribute if you would like, but, I did not have much success in initially getting custom attributes to appear. This would be the more long term approach though, as if we are using Okta, that WOULD be a custom attribute.







From there, the employee would authenticate against DEP with their full email address, and then their username would automatically provision with the prefix of their email, pre populate their password, and populate their full name. 







## Okta - SAML Application







Please see the previous documentation I wrote [here](https://andrewdoering.org/blog/2019/05/01/airwatch-workspace-one-using-saml-and-ldap-with-okta/#8-okta---saml-app-for-airwatch). This page should still work great based on our existing setup. 







## Our room for improvement







We are still working on getting Azure AD set up, so this guide is mainly geared towards a macOS only environment. Though, with the user settings the way they are I am hopeful that this will work.







If it does not, we just have to set up our Organizational Groups to split Windows and macOS devices in o two different buckets, and have a different set of settings between them.  I am looking forward to the challenge when we get there. 







Without having tested group changes thoroughly before writing this, I am worried that the group memberships won't update based on the `Last Modified` attribute not having a value, however, that will remain to be seen as we keep moving forward with this project. 







To also compare this blog post to what VMware originally wrote (at the time of writing this, I have attached the [current KB article](https://kb.vmware.com/s/article/2961230?lang=en_US) in PDF format below:







[Configuring-Okta-LDAP-for-use-with-VMware-Workspace-ONE-UEM-2961230](/assets/blog/2019/12/Configuring-Okta-LDAP-for-use-with-VMware-Workspace-ONE-UEM-2961230.pdf)






## Comments, Questions?







I will likely be a lot more responsive on the MacAdmins slack, you can find me in the #airwatch channel with the username @heimdall. It’s a great resource for people and I highly recommend that you join if you are looking for help with anything Mac related. 







Otherwise, feel free to comment below.



